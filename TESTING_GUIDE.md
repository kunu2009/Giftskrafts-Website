# 🧪 Testing Guide - Product Detail & Review System

## Quick Start Testing

### 1. Test Shop Icon (Mobile Dock)
1. Open website in browser
2. Resize window to mobile view (< 768px width) OR open on mobile device
3. Look at bottom navigation dock
4. **Expected**: Shop icon should display your GK logo PNG
5. **Fallback**: If image doesn't load, you'll see 🛍️ emoji

### 2. Test Categories Navigation
1. Click the **Categories** tab in mobile dock (📂 icon)
   - OR click "Categories" in top navigation
2. **Expected**: See category grid with Valentine, Birthday, Festival, etc.
3. Click any category card (e.g., "Valentine Gifts")
4. **Expected**: Navigate to Shop page with filtered products

### 3. Test Product Detail Page

#### **Open Product Detail:**
1. Go to Shop section
2. Click anywhere on a product card
3. **Expected**: Full-screen modal opens with product details

#### **Image Gallery:**
1. Look for thumbnail images below main image
2. Click different thumbnails
3. **Expected**: Main image changes
4. Click **< (prev)** and **> (next)** arrows
5. **Expected**: Images cycle through

#### **Product Information:**
1. Check product name, price displayed
2. Look for star rating (⭐⭐⭐⭐⭐)
3. Check "About this item" description
4. Check "Specifications" table
5. **Expected**: All information displays clearly

#### **Quantity & Actions:**
1. Click **+** and **-** buttons to change quantity
2. **Expected**: Number increases/decreases (min: 1)
3. Click **Add to Cart** button
4. **Expected**: Modal closes, item added to cart, popup shows
5. Reopen product, click **Buy Now**
6. **Expected**: Redirects to cart page
7. Click **Add to Wishlist** (🤍)
8. **Expected**: Icon changes to ❤️, item saved

### 4. Test Review System

#### **Write a Review:**
1. Open any product detail page
2. Scroll down to "Customer Reviews & Comments"
3. Click on stars (1-5) to select rating
4. **Expected**: Stars fill in with color as you select
5. Enter your name (e.g., "Test User")
6. Enter a comment (e.g., "Great product! Loved it!")
7. Click **Submit Review**
8. **Expected**: 
   - Alert says "Thank you for your review! ⭐"
   - Review appears below in reviews list
   - Average rating updates at top

#### **View Reviews:**
1. Check reviews list section
2. **Expected**:
   - Avatar with first letter of name
   - Name and star rating
   - Comment text
   - Date posted
   - Delete button (only on your own review)

#### **Delete Your Review:**
1. Find a review you wrote (has Delete button)
2. Click **Delete** button
3. Confirm deletion
4. **Expected**: Review removed, rating recalculates

### 5. Test Admin Panel Features

#### **Login to Admin:**
1. Click **⚙️** icon (bottom right)
2. Username: `admin`
3. Password: `gk2025`
4. Click **Login**

#### **Add Product with Full Details:**
1. Go to **Products** tab
2. Fill in form:
   ```
   Product Name: Test Valentine Gift
   Price: 999
   Original Price: 1499
   Category: valentine
   Emoji: 💝
   ```
3. Upload an image (or skip for emoji placeholder)
4. **Additional Images**: Enter comma-separated URLs:
   ```
   https://via.placeholder.com/400/FF6B6B/fff?text=Image1,
   https://via.placeholder.com/400/4ECDC4/fff?text=Image2,
   https://via.placeholder.com/400/45B7D1/fff?text=Image3
   ```
5. **Description**:
   ```
   Perfect Valentine gift set including roses, chocolates, and a greeting card. 
   Show your love with this beautiful combination!
   ```
6. **Specifications** (JSON):
   ```json
   {
     "Brand": "GiftKrafts",
     "Material": "Premium",
     "Includes": "Roses, Chocolates, Card",
     "Color": "Red & Pink",
     "Weight": "500g"
   }
   ```
7. **Stock Status**: In Stock
8. Click **Add Product**
9. **Expected**: Success message, product appears in table

#### **View Product Details:**
1. Go to Shop section
2. Click the product you just added
3. **Expected**:
   - See all 3 images in gallery
   - Discount badge shows "33% OFF"
   - Specifications table shows all fields
   - Description displays fully

#### **Admin Delete Reviews:**
1. Login as admin
2. Open any product with reviews
3. **Expected**: Delete button appears on ALL reviews
4. Click delete on any review
5. **Expected**: Review removes successfully

### 6. Test Firebase Sync (if configured)

#### **Setup:**
1. Open website in two different browsers/devices
2. Login as admin in one

#### **Test Product Sync:**
1. Browser 1: Add a new product
2. Browser 2: Refresh page
3. **Expected**: New product appears immediately

#### **Test Review Sync:**
1. Browser 1: Write a review on a product
2. Browser 2: Open same product detail
3. **Expected**: Review appears without refresh

### 7. Mobile Responsive Testing

#### **On Mobile Device (< 768px):**
1. Open product detail
2. **Expected**:
   - Single column layout
   - Images stack above info
   - All buttons full width
   - Review form responsive
   - Easy to scroll

2. Test navigation dock
3. **Expected**:
   - Visible at bottom
   - Shop icon displays correctly
   - All 6 tabs work

## 🐛 Common Issues & Solutions

### Issue: Shop Icon Not Showing
**Solution**: 
- Check if `gk-logo.png` exists in root folder
- Try renaming to `gk logo.png` (with space)
- Fallback emoji will show if image fails

### Issue: Product Detail Won't Open
**Solution**:
- Open browser console (F12)
- Check for JavaScript errors
- Ensure `openProductDetail()` function exists

### Issue: Reviews Not Saving
**Solution**:
- Check Firebase configuration in console
- Reviews save to localStorage as backup
- Try clearing cache and retry

### Issue: Images Not Loading in Gallery
**Solution**:
- Verify image URLs are valid
- Check if URLs are comma-separated correctly
- Use placeholder URLs for testing

### Issue: Specifications Not Showing
**Solution**:
- Ensure JSON format is valid
- Use JSON validator: https://jsonlint.com
- Example: `{"Brand": "GK", "Size": "Medium"}`

## ✅ Success Criteria

- [x] Shop icon loads in mobile dock
- [x] Categories navigation works
- [x] Product cards clickable
- [x] Product detail modal opens
- [x] Image gallery functional
- [x] Quantity controls work
- [x] Add to cart works
- [x] Buy now redirects
- [x] Wishlist toggle works
- [x] Star rating input responsive
- [x] Reviews submit successfully
- [x] Reviews display with avatars
- [x] Users can delete own reviews
- [x] Admins can delete all reviews
- [x] Average rating calculates correctly
- [x] Admin panel has new fields
- [x] Products save with full details
- [x] Specifications display properly
- [x] Discount badges show correctly
- [x] Firebase sync works (if configured)

## 📊 Test Data Examples

### Sample Product Data:
```javascript
{
  name: "Premium Valentine Gift Set",
  price: 1299,
  originalPrice: 1999,
  emoji: "💝",
  images: [
    "https://via.placeholder.com/400/FF6B6B/fff?text=Rose+Bouquet",
    "https://via.placeholder.com/400/FF69B4/fff?text=Chocolates",
    "https://via.placeholder.com/400/FFB6C1/fff?text=Gift+Box"
  ],
  description: "A complete Valentine's Day gift set featuring premium roses, Belgian chocolates, and a handwritten card.",
  specifications: {
    "Brand": "GiftKrafts",
    "Includes": "12 Red Roses, 200g Chocolates, Greeting Card",
    "Color": "Red & Pink",
    "Occasion": "Valentine's Day",
    "Weight": "800g",
    "Delivery": "Same Day Available"
  },
  stockStatus: "in-stock",
  category: "valentine"
}
```

### Sample Reviews:
```javascript
[
  {
    name: "Rahul Kumar",
    rating: 5,
    comment: "Amazing gift! My girlfriend loved it. The roses were fresh and chocolates were delicious!"
  },
  {
    name: "Priya Sharma",
    rating: 4,
    comment: "Good quality product. Fast delivery. Only 4 stars because packaging could be better."
  },
  {
    name: "Amit Patel",
    rating: 5,
    comment: "Perfect Valentine gift! Highly recommended. Will order again next year."
  }
]
```

## 🎯 Performance Checks

1. **Page Load Time**: Should load in < 3 seconds
2. **Modal Open Speed**: Instant (< 0.3s)
3. **Image Loading**: Progressive loading with placeholders
4. **Review Submission**: < 2 seconds
5. **Firebase Sync**: Real-time (instant)

## 📝 Notes

- Test on different browsers: Chrome, Firefox, Safari, Edge
- Test on different devices: Mobile, Tablet, Desktop
- Test different screen orientations: Portrait & Landscape
- Clear cache if seeing old data
- Use Incognito mode for clean tests

---

**Happy Testing! 🎉**

If you find any issues, check the browser console (F12) for error messages.
