# How to upload images for your **Listing model** using **Multer** and **Cloudinary**:


### **Step 1: Update the `Listing` Model**
Since your `Listing` model already exists, add fields to store the Cloudinary URL and the public ID for each image:

```javascript
const mongoose = require('mongoose')

const listingSchema = new mongoose.Schema({
    streetAddress: {
        type: String,
        required: true,
    },
    city: {
        type: String,
        required: true,
    },
    price: {
        type: Number,
        required: true,
        min: 0,
    },
    size: {
        type: Number,
        required: true,
        min: 0,
    },
    image: {
      url: { type: String, required: true }, // Cloudinary URL
      cloudinary_id: { type: String, required: true }, // Public ID for deletion
    },
    owner: {
        type: mongoose.Schema.Types.ObjectId,
        ref: 'User',
    },
    favoritedByUsers: [
        {
            type: mongoose.Schema.Types.ObjectId,
            ref: 'User',   
        }
    ],
}, {timestamps: true})

const Listing = mongoose.model('Listing', listingSchema)

module.exports = Listing
```

---

### **Step 2: Configure Cloudinary**
1. **Install Dependencies**:
   If you haven’t already:
   ```bash
   npm install cloudinary multer multer-storage-cloudinary
   ```

2. **[Sign Up for a Cloudinary Account](sign-up-for-cloudinary.md)**
   Ensure your `.env` file contains:
   ```plaintext
   CLOUDINARY_CLOUD_NAME=your_cloud_name
   CLOUDINARY_API_KEY=your_api_key
   CLOUDINARY_API_SECRET=your_api_secret
   ```
  
3.  **Set Up Cloudinary Configuration**:
   Add a `config/cloudinary.js` file:
       ```javascript
       const cloudinary = require("cloudinary").v2;
       const dotenv = require("dotenv");
    
       dotenv.config();
    
       cloudinary.config({
         cloud_name: process.env.CLOUDINARY_CLOUD_NAME,
         api_key: process.env.CLOUDINARY_API_KEY,
         api_secret: process.env.CLOUDINARY_API_SECRET,
       });
    
       module.exports = cloudinary;
       ```

---

### **Step 3: Configure Multer with Cloudinary**
1. Add a `config/multer.js` file.

2. Create a storage configuration for Multer using `multer-storage-cloudinary`:
   ```javascript
   const multer = require("multer");
   const { CloudinaryStorage } = require("multer-storage-cloudinary");
   const cloudinary = require("../config/cloudinary");

   const storage = new CloudinaryStorage({
     cloudinary: cloudinary,
     params: {
       folder: "listings", // Cloudinary folder name
       allowed_formats: ["jpg", "jpeg", "png"], // Allowed file types
     },
   });

   const upload = multer({ storage: storage });

   module.exports = upload;
   ```
---

### **Step 4: Add a Route for Image Uploads**
1. In your `listings.controller`, modify the function for creating listings with image uploads.
    #### **Create a New Listing with an Image**
    
    ```javascript
    const createListing = async (req, res) => {
        console.log(req.file)
        try {
            // req.body.owner = req.session.user._id
            req.body.imgUrl = {
                url: req.file.path, // Cloudinary URL
                cloudinary_id: req.file.filename, // Cloudinary public ID
            }
            req.body.owner = req.params.userId
            await Listing.create(req.body)
            res.redirect('/listings')
        } catch (error) {
            console.log(error)
    
        }
    }
    ```

2. In your `server.js` file, import Multer.

    ```js
    const upload = require("../config/multer"); // Import Multer
    ```
3. Use Multer's middleware in your route for posting listing form data.

    ```js
    app.post('/listings/:userId', upload.single("imgUrl"), listingsCtrl.createListing)
    ```
    
---

### **Step 5: Update Your Frontend**
1. In `views/listings/new.ejs` ensure your form includes a file input for uploading images:
   ```html
        <form action="/listings/<%= user._id %>" method="POST" enctype="multipart/form-data">
            <label for="">Street Address: </label>
            <input type="text" name="streetAddress">
            <label for="">City: </label>
            <input type="text" name="city">
            <label for="">Price: </label>
            <input type="number" name="price">
            <label for="">Size: </label>
            <input type="text" name="size">
            <label for="">Image: </label>
            <!-- <input type="text" name="imgUrl"> -->
            <input type="file" name="imgUrl" accept="image/*" required />
            <button type="submit" class="btn btn-primary">Add Listing</button>
        </form>
   ```

2. Edit your `views/listings/index.ejs` and `views/listings/show.ejs` to display the uploaded image url from Multer:
    ```html
    <div class="card mt-2" style="width: 18rem;">
        <!-- change the img src below from listing.imgUrl to listing.imgUrl.url -->
        <img src="<%= listing.imgUrl.url %>" class="card-img-top" alt="listing for <%= listing.streetAddress %>">
        <div class="card-body">
            <h5 class="card-title">$<%= listing.price %></h5>
            <p class="card-text"><%= listing.streetAddress %></p>
            <a href="/listings/<%= listing._id %>" class="btn btn-primary">View Details</a>
        </div>
    </div>
    ```



---

#### **Update an Existing Listing with an Image**
If you need to allow updates to a listing’s image:

1. Use Multer's middleware in your `put` route for updating listing form data.
    ```js
    app.put('/listings/:userId/:listingId', upload.single("imgUrl"), listingsCtrl.update)
    ```
    
2. In your `listings.controller`, modify the function for updating listings with image uploads.
    ```javascript
    const cloudinary = require("../config/cloudinary") // require Cloudinary in the controller file

    router.put("/listings/:id", upload.single("image"), async (req, res) => {
      try {
        const listing = await Listing.findById(req.params.id);
        if (!listing) return res.status(404).json({ message: "Listing not found" });
    
        // Delete the old image from Cloudinary
        await cloudinary.uploader.destroy(listing.image.cloudinary_id);
    
        // Update listing with the new image
        listing.title = req.body.title || listing.title;
        listing.description = req.body.description || listing.description;
        listing.price = req.body.price || listing.price;
        listing.image = {
          url: req.file.path, // New Cloudinary URL
          cloudinary_id: req.file.filename, // New Cloudinary public ID
        };
    
        const updatedListing = await listing.save();
        res.json(updatedListing);
      } catch (error) {
        res.status(500).json({ message: error.message });
      }
    });
    ```
    
3. In `views/listings/edit.ejs` ensure your form includes a file input type for uploading images and enctype in the `form` tag:
   ```html
        <form action="/listings/<%= user._id %>/<%= listing._id %>?_method=PUT" method="POST" enctype="multipart/form-data">
            <label for="">Street Address: </label>
            <input type="text" name="streetAddress" value="<%= listing.streetAddress %>">
            <label for="">City: </label>
            <input type="text" name="city" value="<%= listing.city %>">
            <label for="">Price: </label>
            <input type="number" name="price" value="<%= listing.price %>">
            <label for="">Size: </label>
            <input type="text" name="size" value="<%= listing.size %>">
            <label for="">Image: </label>
            <input type="file" name="imgUrl" accept="image/*" required />
            <button type="submit" class="btn btn-primary">Edit Listing</button>
        </form>
   ```
   

---

### **Step 6: Test Your Application**
1. Start your server:
   ```bash
   npm start
   ```

2. Use Postman or your frontend to send a `POST` request to create a listing with an image:
   - Endpoint: `POST /listings`
   - Body: `form-data` with fields `title`, `description`, `price`, and `image` (as a file).

3. Test the `PUT /listings/:id` route to update an image.

---

### **Step 7: Handle Image Deletion (Optional)**
Add a route to delete a listing along with its image:
```javascript
router.delete("/listings/:id", async (req, res) => {
  try {
    const listing = await Listing.findById(req.params.id);
    if (!listing) return res.status(404).json({ message: "Listing not found" });

    // Delete image from Cloudinary
    const cloudinary = require("../config/cloudinary");
    await cloudinary.uploader.destroy(listing.image.cloudinary_id);

    // Delete listing from database
    await listing.remove();
    res.json({ message: "Listing and image deleted" });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});
```

---

### **Step 8: Verify Everything Works**
- Test creating listings with images.
- Test updating a listing with a new image.
- Test deleting a listing and its image.

With these changes, users can now upload and manage images for their listings in your MEN stack app! 🎉
