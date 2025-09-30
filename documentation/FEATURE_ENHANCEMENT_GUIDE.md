# 💡 Feature Enhancement Roadmap - Giftskrafts by GK

## 🎯 Essential Features Implementation Guide

### 🔐 Phase 1: User Account & Authentication System

#### User Registration & Login
```html
<!-- Add to index.html after the existing nav -->
<div id="authModal" class="modal" style="display: none;">
    <div class="modal-content">
        <span class="close" onclick="closeAuthModal()">&times;</span>
        
        <!-- Login Form -->
        <div id="loginForm" class="auth-form">
            <h2>Welcome Back!</h2>
            <form onsubmit="handleLogin(event)">
                <div class="form-group">
                    <label>Email:</label>
                    <input type="email" id="loginEmail" required>
                </div>
                <div class="form-group">
                    <label>Password:</label>
                    <input type="password" id="loginPassword" required>
                </div>
                <button type="submit" class="btn btn-primary">Login</button>
                <p>Don't have an account? <a href="#" onclick="showRegisterForm()">Register here</a></p>
            </form>
        </div>
        
        <!-- Register Form -->
        <div id="registerForm" class="auth-form" style="display: none;">
            <h2>Create Account</h2>
            <form onsubmit="handleRegister(event)">
                <div class="form-group">
                    <label>Full Name:</label>
                    <input type="text" id="registerName" required>
                </div>
                <div class="form-group">
                    <label>Email:</label>
                    <input type="email" id="registerEmail" required>
                </div>
                <div class="form-group">
                    <label>Phone:</label>
                    <input type="tel" id="registerPhone" required>
                </div>
                <div class="form-group">
                    <label>Password:</label>
                    <input type="password" id="registerPassword" required>
                </div>
                <button type="submit" class="btn btn-primary">Register</button>
                <p>Already have an account? <a href="#" onclick="showLoginForm()">Login here</a></p>
            </form>
        </div>
    </div>
</div>

<!-- User Profile Dropdown -->
<div id="userProfileDropdown" class="profile-dropdown" style="display: none;">
    <div class="dropdown-content">
        <a href="#" onclick="showUserProfile()">👤 My Profile</a>
        <a href="#" onclick="showOrderHistory()">📦 My Orders</a>
        <a href="#" onclick="showWishlist()">❤️ Wishlist</a>
        <a href="#" onclick="logout()">🚪 Logout</a>
    </div>
</div>
```

#### JavaScript Implementation
```javascript
// Add to your existing JavaScript

// User Authentication System
class UserAuth {
    constructor() {
        this.currentUser = null;
        this.checkUserState();
    }
    
    async register(userData) {
        try {
            if (window.firebaseDB.useFirebase) {
                // Firebase registration
                const userCredential = await firebase.auth().createUserWithEmailAndPassword(
                    userData.email, 
                    userData.password
                );
                
                // Save user data to Firestore
                await firebaseDB.addDoc(firebaseDB.collection(firebaseDB.db, 'users'), {
                    uid: userCredential.user.uid,
                    name: userData.name,
                    email: userData.email,
                    phone: userData.phone,
                    createdAt: new Date(),
                    addresses: [],
                    wishlist: []
                });
                
                this.currentUser = userCredential.user;
                this.updateUI();
                return { success: true };
            } else {
                // Local storage fallback
                const users = JSON.parse(localStorage.getItem('users') || '[]');
                const newUser = {
                    id: Date.now().toString(),
                    ...userData,
                    createdAt: new Date()
                };
                users.push(newUser);
                localStorage.setItem('users', JSON.stringify(users));
                localStorage.setItem('currentUser', JSON.stringify(newUser));
                this.currentUser = newUser;
                this.updateUI();
                return { success: true };
            }
        } catch (error) {
            return { success: false, error: error.message };
        }
    }
    
    async login(email, password) {
        try {
            if (window.firebaseDB.useFirebase) {
                const userCredential = await firebase.auth().signInWithEmailAndPassword(email, password);
                this.currentUser = userCredential.user;
                this.updateUI();
                return { success: true };
            } else {
                const users = JSON.parse(localStorage.getItem('users') || '[]');
                const user = users.find(u => u.email === email && u.password === password);
                if (user) {
                    localStorage.setItem('currentUser', JSON.stringify(user));
                    this.currentUser = user;
                    this.updateUI();
                    return { success: true };
                } else {
                    return { success: false, error: 'Invalid credentials' };
                }
            }
        } catch (error) {
            return { success: false, error: error.message };
        }
    }
    
    logout() {
        if (window.firebaseDB.useFirebase) {
            firebase.auth().signOut();
        } else {
            localStorage.removeItem('currentUser');
        }
        this.currentUser = null;
        this.updateUI();
    }
    
    checkUserState() {
        if (window.firebaseDB.useFirebase) {
            firebase.auth().onAuthStateChanged((user) => {
                this.currentUser = user;
                this.updateUI();
            });
        } else {
            const user = localStorage.getItem('currentUser');
            if (user) {
                this.currentUser = JSON.parse(user);
                this.updateUI();
            }
        }
    }
    
    updateUI() {
        const loginBtn = document.getElementById('loginBtn');
        const userProfile = document.getElementById('userProfile');
        
        if (this.currentUser) {
            if (loginBtn) loginBtn.style.display = 'none';
            if (userProfile) {
                userProfile.style.display = 'block';
                userProfile.innerHTML = `
                    <span onclick="toggleProfileDropdown()" class="user-name">
                        Hi, ${this.currentUser.displayName || this.currentUser.name || 'User'}! 
                        <i class="fas fa-chevron-down"></i>
                    </span>
                `;
            }
        } else {
            if (loginBtn) loginBtn.style.display = 'block';
            if (userProfile) userProfile.style.display = 'none';
        }
    }
}

// Initialize user auth
const userAuth = new UserAuth();

// Auth modal functions
function showAuthModal() {
    document.getElementById('authModal').style.display = 'block';
}

function closeAuthModal() {
    document.getElementById('authModal').style.display = 'none';
}

function showLoginForm() {
    document.getElementById('loginForm').style.display = 'block';
    document.getElementById('registerForm').style.display = 'none';
}

function showRegisterForm() {
    document.getElementById('loginForm').style.display = 'none';
    document.getElementById('registerForm').style.display = 'block';
}

async function handleLogin(event) {
    event.preventDefault();
    const email = document.getElementById('loginEmail').value;
    const password = document.getElementById('loginPassword').value;
    
    const result = await userAuth.login(email, password);
    if (result.success) {
        closeAuthModal();
        showNotification('Login successful! Welcome back!', 'success');
    } else {
        showNotification(result.error, 'error');
    }
}

async function handleRegister(event) {
    event.preventDefault();
    const userData = {
        name: document.getElementById('registerName').value,
        email: document.getElementById('registerEmail').value,
        phone: document.getElementById('registerPhone').value,
        password: document.getElementById('registerPassword').value
    };
    
    const result = await userAuth.register(userData);
    if (result.success) {
        closeAuthModal();
        showNotification('Registration successful! Welcome to Giftskrafts!', 'success');
    } else {
        showNotification(result.error, 'error');
    }
}

function logout() {
    userAuth.logout();
    showNotification('Logged out successfully!', 'success');
    toggleProfileDropdown(); // Close dropdown
}
```

### 🛒 Phase 2: Enhanced Shopping Cart System

#### Persistent Cart with Local Storage + Firebase
```javascript
// Enhanced Shopping Cart
class EnhancedShoppingCart extends ShoppingCart {
    constructor() {
        super();
        this.setupEventListeners();
    }
    
    addItem(product, quantity = 1, variant = null) {
        const cartItem = {
            id: product.id,
            name: product.name,
            price: product.price,
            image: product.image,
            quantity: quantity,
            variant: variant, // For size, color, etc.
            addedAt: new Date(),
            category: product.category
        };
        
        const existingItemIndex = this.items.findIndex(item => 
            item.id === product.id && 
            JSON.stringify(item.variant) === JSON.stringify(variant)
        );
        
        if (existingItemIndex >= 0) {
            this.items[existingItemIndex].quantity += quantity;
        } else {
            this.items.push(cartItem);
        }
        
        this.saveCart();
        this.updateCartUI();
        this.showCartNotification(`${product.name} added to cart!`);
        this.trackAddToCart(cartItem);
    }
    
    trackAddToCart(item) {
        // Analytics tracking
        if (typeof gtag !== 'undefined') {
            gtag('event', 'add_to_cart', {
                currency: 'INR',
                value: item.price,
                items: [{
                    item_id: item.id,
                    item_name: item.name,
                    category: item.category,
                    quantity: item.quantity,
                    price: item.price
                }]
            });
        }
    }
    
    async syncWithServer() {
        if (userAuth.currentUser && window.firebaseDB.useFirebase) {
            try {
                await firebaseDB.updateDoc(
                    firebaseDB.doc(firebaseDB.db, 'users', userAuth.currentUser.uid),
                    { cart: this.items }
                );
            } catch (error) {
                console.log('Cart sync failed:', error);
            }
        }
    }
    
    setupEventListeners() {
        // Auto-save cart every 30 seconds
        setInterval(() => {
            this.syncWithServer();
        }, 30000);
        
        // Save cart when user leaves page
        window.addEventListener('beforeunload', () => {
            this.syncWithServer();
        });
    }
    
    renderCartSidebar() {
        const cartSidebar = `
            <div id="cartSidebar" class="cart-sidebar">
                <div class="cart-header">
                    <h3>Shopping Cart (${this.getTotalItems()})</h3>
                    <button onclick="closeCartSidebar()" class="close-btn">&times;</button>
                </div>
                <div class="cart-body">
                    ${this.items.length === 0 ? 
                        '<p class="empty-cart-message">Your cart is empty</p>' :
                        this.items.map(item => this.renderSidebarItem(item)).join('')
                    }
                </div>
                ${this.items.length > 0 ? `
                    <div class="cart-footer">
                        <div class="cart-total">
                            <strong>Total: ₹${this.getTotal().toFixed(2)}</strong>
                        </div>
                        <button onclick="proceedToCheckout()" class="btn btn-primary">
                            Checkout
                        </button>
                        <button onclick="showCartPage()" class="btn btn-secondary">
                            View Cart
                        </button>
                    </div>
                ` : ''}
            </div>
            <div id="cartOverlay" class="cart-overlay" onclick="closeCartSidebar()"></div>
        `;
        
        // Remove existing sidebar
        const existing = document.getElementById('cartSidebar');
        if (existing) existing.remove();
        
        // Add new sidebar
        document.body.insertAdjacentHTML('beforeend', cartSidebar);
    }
    
    renderSidebarItem(item) {
        return `
            <div class="sidebar-cart-item">
                <img src="${item.image}" alt="${item.name}" class="item-thumb">
                <div class="item-details">
                    <div class="item-name">${item.name}</div>
                    <div class="item-price">₹${item.price} × ${item.quantity}</div>
                    ${item.variant ? `<div class="item-variant">${JSON.stringify(item.variant)}</div>` : ''}
                </div>
                <button onclick="removeFromCart('${item.id}')" class="remove-btn">×</button>
            </div>
        `;
    }
}

// Cart sidebar functions
function showCartSidebar() {
    cart.renderCartSidebar();
    document.getElementById('cartSidebar').classList.add('active');
    document.getElementById('cartOverlay').classList.add('active');
}

function closeCartSidebar() {
    document.getElementById('cartSidebar').classList.remove('active');
    document.getElementById('cartOverlay').classList.remove('active');
}

// Replace the old cart with enhanced version
const cart = new EnhancedShoppingCart();
```

### ❤️ Phase 3: Advanced Wishlist System

#### Wishlist with Social Sharing
```javascript
class AdvancedWishlist extends Wishlist {
    constructor(userId = null) {
        super(userId);
        this.setupSocialSharing();
    }
    
    addItem(product) {
        if (!this.items.find(item => item.id === product.id)) {
            const wishlistItem = {
                ...product,
                addedAt: new Date(),
                notes: '',
                priority: 'medium' // low, medium, high
            };
            
            this.items.push(wishlistItem);
            this.saveWishlist();
            this.updateWishlistUI();
            this.showNotification('Added to wishlist! ❤️', 'success');
            this.trackWishlistAdd(product);
        } else {
            this.showNotification('Already in your wishlist!', 'info');
        }
    }
    
    shareWishlist() {
        const wishlistUrl = this.generateShareableUrl();
        const shareText = "Check out my wishlist on Giftskrafts by GK!";
        
        if (navigator.share) {
            navigator.share({
                title: 'My Wishlist - Giftskrafts by GK',
                text: shareText,
                url: wishlistUrl
            });
        } else {
            // Fallback to copying link
            navigator.clipboard.writeText(wishlistUrl);
            this.showNotification('Wishlist link copied to clipboard!', 'success');
        }
    }
    
    generateShareableUrl() {
        const baseUrl = window.location.origin;
        const wishlistId = this.createShareableWishlist();
        return `${baseUrl}?wishlist=${wishlistId}`;
    }
    
    createShareableWishlist() {
        const shareableWishlist = {
            id: 'wl_' + Date.now(),
            items: this.items,
            createdBy: userAuth.currentUser?.name || 'Anonymous',
            createdAt: new Date(),
            public: true
        };
        
        // Save to Firebase or localStorage
        if (window.firebaseDB.useFirebase) {
            firebaseDB.addDoc(
                firebaseDB.collection(firebaseDB.db, 'sharedWishlists'),
                shareableWishlist
            );
        } else {
            const sharedWishlists = JSON.parse(localStorage.getItem('sharedWishlists') || '[]');
            sharedWishlists.push(shareableWishlist);
            localStorage.setItem('sharedWishlists', JSON.stringify(sharedWishlists));
        }
        
        return shareableWishlist.id;
    }
    
    renderWishlistPage() {
        return `
            <div class="container">
                <div class="wishlist-header">
                    <h1>My Wishlist <span class="item-count">(${this.items.length} items)</span></h1>
                    <div class="wishlist-actions">
                        <button onclick="wishlist.shareWishlist()" class="btn btn-secondary">
                            <i class="fas fa-share"></i> Share Wishlist
                        </button>
                        <button onclick="wishlist.addAllToCart()" class="btn btn-primary">
                            <i class="fas fa-cart-plus"></i> Add All to Cart
                        </button>
                    </div>
                </div>
                
                ${this.items.length === 0 ? this.renderEmptyWishlist() : this.renderWishlistGrid()}
            </div>
        `;
    }
    
    renderEmptyWishlist() {
        return `
            <div class="empty-wishlist">
                <div class="empty-icon">💝</div>
                <h2>Your wishlist is empty</h2>
                <p>Save your favorite gifts to buy them later!</p>
                <button onclick="showHomePage()" class="btn btn-primary">Start Shopping</button>
            </div>
        `;
    }
    
    renderWishlistGrid() {
        return `
            <div class="wishlist-grid">
                ${this.items.map(item => this.renderAdvancedWishlistItem(item)).join('')}
            </div>
        `;
    }
    
    renderAdvancedWishlistItem(item) {
        return `
            <div class="wishlist-card" data-id="${item.id}">
                <div class="item-image">
                    <img src="${item.image}" alt="${item.name}">
                    <div class="item-overlay">
                        <button onclick="quickViewProduct('${item.id}')" class="quick-view-btn">
                            <i class="fas fa-eye"></i>
                        </button>
                    </div>
                </div>
                <div class="item-info">
                    <h3 class="item-name">${item.name}</h3>
                    <p class="item-price">₹${item.price}</p>
                    <div class="item-meta">
                        <span class="added-date">Added ${this.formatDate(item.addedAt)}</span>
                        <span class="priority priority-${item.priority}">${item.priority}</span>
                    </div>
                    <div class="item-actions">
                        <button onclick="addToCartFromWishlist('${item.id}')" class="btn btn-primary">
                            <i class="fas fa-cart-plus"></i> Add to Cart
                        </button>
                        <button onclick="removeFromWishlist('${item.id}')" class="btn btn-outline">
                            <i class="fas fa-trash"></i>
                        </button>
                        <button onclick="moveToCart('${item.id}')" class="btn btn-secondary">
                            <i class="fas fa-arrow-right"></i> Move to Cart
                        </button>
                    </div>
                </div>
            </div>
        `;
    }
    
    addAllToCart() {
        this.items.forEach(item => {
            cart.addItem(item, 1);
        });
        this.showNotification(`${this.items.length} items added to cart!`, 'success');
    }
    
    moveToCart(productId) {
        const item = this.items.find(item => item.id === productId);
        if (item) {
            cart.addItem(item, 1);
            this.removeItem(productId);
            this.showNotification(`${item.name} moved to cart!`, 'success');
        }
    }
    
    formatDate(date) {
        const now = new Date();
        const diffTime = Math.abs(now - new Date(date));
        const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24));
        
        if (diffDays === 1) return 'yesterday';
        if (diffDays < 7) return `${diffDays} days ago`;
        if (diffDays < 30) return `${Math.ceil(diffDays / 7)} weeks ago`;
        return `${Math.ceil(diffDays / 30)} months ago`;
    }
    
    trackWishlistAdd(product) {
        if (typeof gtag !== 'undefined') {
            gtag('event', 'add_to_wishlist', {
                currency: 'INR',
                value: product.price,
                items: [{
                    item_id: product.id,
                    item_name: product.name,
                    category: product.category,
                    price: product.price
                }]
            });
        }
    }
}

// Replace wishlist with advanced version
const wishlist = new AdvancedWishlist(userAuth.currentUser?.uid);
```

### 💳 Phase 4: Payment Gateway Integration

#### Multiple Payment Options
```javascript
class PaymentGateway {
    constructor() {
        this.razorpayKey = 'rzp_test_your_key_here'; // Replace with actual key
        this.setupPaymentMethods();
    }
    
    setupPaymentMethods() {
        this.paymentMethods = {
            razorpay: {
                name: 'Online Payment',
                description: 'Pay with Cards, UPI, Net Banking, Wallets',
                fee: 0,
                enabled: true
            },
            cod: {
                name: 'Cash on Delivery',
                description: 'Pay when you receive your order',
                fee: 25,
                enabled: true
            },
            upi: {
                name: 'UPI Direct',
                description: 'Pay directly via UPI apps',
                fee: 0,
                enabled: true
            }
        };
    }
    
    async processPayment(orderData, paymentMethod) {
        switch (paymentMethod) {
            case 'razorpay':
                return await this.processRazorpayPayment(orderData);
            case 'cod':
                return await this.processCODPayment(orderData);
            case 'upi':
                return await this.processUPIPayment(orderData);
            default:
                throw new Error('Invalid payment method');
        }
    }
    
    async processRazorpayPayment(orderData) {
        return new Promise((resolve, reject) => {
            const options = {
                key: this.razorpayKey,
                amount: orderData.amount * 100, // Amount in paise
                currency: 'INR',
                name: 'Giftskrafts by GK',
                description: 'Beautiful Gifts for Special Moments',
                image: '/gk-logo.png',
                order_id: orderData.razorpayOrderId, // Create this on server
                handler: async (response) => {
                    try {
                        const result = await this.verifyPayment(response, orderData);
                        resolve(result);
                    } catch (error) {
                        reject(error);
                    }
                },
                prefill: {
                    name: orderData.customerName,
                    email: orderData.customerEmail,
                    contact: orderData.customerPhone
                },
                notes: {
                    order_id: orderData.orderId,
                    customer_id: userAuth.currentUser?.uid || 'guest'
                },
                theme: {
                    color: '#7b2cbf'
                },
                modal: {
                    ondismiss: () => {
                        reject(new Error('Payment cancelled by user'));
                    }
                },
                retry: {
                    enabled: true,
                    max_count: 3
                }
            };
            
            const rzp = new Razorpay(options);
            rzp.on('payment.failed', (response) => {
                reject(new Error(response.error.description));
            });
            
            rzp.open();
        });
    }
    
    async processCODPayment(orderData) {
        // For COD, we just need to create the order
        const order = {
            ...orderData,
            paymentMethod: 'cod',
            paymentStatus: 'pending',
            orderStatus: 'confirmed',
            paymentId: 'COD_' + Date.now()
        };
        
        return { success: true, order };
    }
    
    async processUPIPayment(orderData) {
        // Generate UPI payment link
        const upiUrl = this.generateUPIUrl(orderData);
        
        // Show UPI options
        this.showUPIOptions(upiUrl, orderData);
        
        return new Promise((resolve) => {
            // For demo purposes, resolve after 30 seconds
            // In real implementation, you'd check payment status
            setTimeout(() => {
                resolve({ 
                    success: true, 
                    paymentId: 'UPI_' + Date.now(),
                    order: orderData 
                });
            }, 30000);
        });
    }
    
    generateUPIUrl(orderData) {
        const upiId = 'giftskrafts@paytm'; // Your UPI ID
        const amount = orderData.amount;
        const note = `Payment for Order ${orderData.orderId}`;
        
        return `upi://pay?pa=${upiId}&am=${amount}&cu=INR&tn=${encodeURIComponent(note)}`;
    }
    
    showUPIOptions(upiUrl, orderData) {
        const modal = document.createElement('div');
        modal.className = 'upi-modal';
        modal.innerHTML = `
            <div class="upi-content">
                <h3>Pay with UPI</h3>
                <div class="upi-amount">₹${orderData.amount}</div>
                <div class="upi-apps">
                    <a href="paytmmp://pay?pa=giftskrafts@paytm&am=${orderData.amount}" class="upi-app">
                        <img src="/icons/paytm.png" alt="Paytm">
                        Paytm
                    </a>
                    <a href="phonepe://pay?pa=giftskrafts@paytm&am=${orderData.amount}" class="upi-app">
                        <img src="/icons/phonepe.png" alt="PhonePe">
                        PhonePe
                    </a>
                    <a href="gpay://upi/pay?pa=giftskrafts@paytm&am=${orderData.amount}" class="upi-app">
                        <img src="/icons/gpay.png" alt="Google Pay">
                        Google Pay
                    </a>
                </div>
                <div class="upi-qr">
                    <div id="qrcode"></div>
                    <p>Scan this QR code with any UPI app</p>
                </div>
                <button onclick="closeUPIModal()" class="btn btn-secondary">Cancel</button>
            </div>
        `;
        
        document.body.appendChild(modal);
        
        // Generate QR code (you'll need to include a QR code library)
        // new QRCode(document.getElementById("qrcode"), upiUrl);
    }
    
    async verifyPayment(response, orderData) {
        // In a real implementation, verify payment on server
        // For now, we'll simulate verification
        return {
            success: true,
            paymentId: response.razorpay_payment_id,
            orderId: response.razorpay_order_id,
            signature: response.razorpay_signature
        };
    }
    
    renderPaymentMethods(total) {
        return Object.entries(this.paymentMethods)
            .filter(([key, method]) => method.enabled)
            .map(([key, method]) => `
                <label class="payment-method">
                    <input type="radio" name="paymentMethod" value="${key}" ${key === 'razorpay' ? 'checked' : ''}>
                    <div class="method-info">
                        <div class="method-name">${method.name}</div>
                        <div class="method-description">${method.description}</div>
                        ${method.fee > 0 ? `<div class="method-fee">+ ₹${method.fee} fee</div>` : ''}
                    </div>
                    <div class="method-total">
                        ₹${(total + method.fee).toFixed(2)}
                    </div>
                </label>
            `).join('');
    }
}

// Initialize payment gateway
const paymentGateway = new PaymentGateway();
```

### 📊 Phase 5: Analytics & Tracking

#### Comprehensive Analytics Setup
```javascript
class AdvancedAnalytics {
    constructor() {
        this.initializeTracking();
        this.setupEventListeners();
    }
    
    initializeTracking() {
        // Google Analytics 4
        this.setupGA4();
        
        // Facebook Pixel
        this.setupFacebookPixel();
        
        // Custom analytics
        this.sessionData = {
            sessionId: this.generateSessionId(),
            startTime: Date.now(),
            pageViews: 0,
            events: []
        };
    }
    
    setupGA4() {
        // Configure enhanced ecommerce
        gtag('config', 'G-YOUR-GA4-ID', {
            custom_map: {
                'custom_parameter_1': 'gift_category',
                'custom_parameter_2': 'customer_type'
            }
        });
        
        // Set user properties
        if (userAuth.currentUser) {
            gtag('set', {
                'user_id': userAuth.currentUser.uid,
                'customer_type': 'registered'
            });
        }
    }
    
    setupFacebookPixel() {
        fbq('init', 'YOUR_PIXEL_ID');
        fbq('track', 'PageView');
        
        // Set up automatic events
        fbq('track', 'ViewContent');
    }
    
    trackEvent(eventName, parameters = {}) {
        // Add session data
        const eventData = {
            ...parameters,
            session_id: this.sessionData.sessionId,
            timestamp: Date.now(),
            user_id: userAuth.currentUser?.uid || 'anonymous'
        };
        
        // Google Analytics
        gtag('event', eventName, eventData);
        
        // Facebook Pixel
        fbq('track', eventName, eventData);
        
        // Custom tracking
        this.sessionData.events.push({
            event: eventName,
            data: eventData
        });
        
        // Save to database if needed
        this.saveEventData(eventName, eventData);
    }
    
    trackPurchase(orderData) {
        const purchaseData = {
            transaction_id: orderData.orderId,
            value: orderData.amount,
            currency: 'INR',
            coupon: orderData.couponCode || '',
            payment_type: orderData.paymentMethod,
            items: orderData.items.map(item => ({
                item_id: item.id,
                item_name: item.name,
                category: item.category || 'gifts',
                quantity: item.quantity,
                price: item.price
            }))
        };
        
        // GA4 Purchase
        gtag('event', 'purchase', purchaseData);
        
        // Facebook Purchase
        fbq('track', 'Purchase', {
            value: orderData.amount,
            currency: 'INR',
            content_ids: orderData.items.map(item => item.id),
            content_type: 'product'
        });
        
        // Custom purchase tracking
        this.trackEvent('custom_purchase', {
            order_value: orderData.amount,
            item_count: orderData.items.length,
            customer_type: userAuth.currentUser ? 'registered' : 'guest',
            payment_method: orderData.paymentMethod
        });
    }
    
    trackProductView(product) {
        this.trackEvent('view_item', {
            currency: 'INR',
            value: product.price,
            items: [{
                item_id: product.id,
                item_name: product.name,
                category: product.category || 'gifts',
                price: product.price
            }]
        });
    }
    
    trackSearchResults(searchTerm, resultCount) {
        this.trackEvent('search', {
            search_term: searchTerm,
            result_count: resultCount
        });
    }
    
    trackUserBehavior() {
        // Scroll depth tracking
        let maxScroll = 0;
        let scrollTimer;
        
        window.addEventListener('scroll', () => {
            clearTimeout(scrollTimer);
            scrollTimer = setTimeout(() => {
                const scrollPercent = Math.round(
                    (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100
                );
                
                if (scrollPercent > maxScroll && scrollPercent % 25 === 0) {
                    maxScroll = scrollPercent;
                    this.trackEvent('scroll', { scroll_depth: scrollPercent });
                }
            }, 250);
        });
        
        // Time on page
        const startTime = Date.now();
        window.addEventListener('beforeunload', () => {
            const timeSpent = Math.round((Date.now() - startTime) / 1000);
            this.trackEvent('timing_complete', {
                name: 'time_on_page',
                value: timeSpent
            });
        });
        
        // Click tracking
        document.addEventListener('click', (e) => {
            if (e.target.matches('[data-track]')) {
                const trackingData = JSON.parse(e.target.dataset.track);
                this.trackEvent('click', trackingData);
            }
        });
    }
    
    generateSessionId() {
        return 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
    }
    
    async saveEventData(eventName, eventData) {
        if (window.firebaseDB.useFirebase) {
            try {
                await firebaseDB.addDoc(
                    firebaseDB.collection(firebaseDB.db, 'analytics'),
                    {
                        event: eventName,
                        data: eventData,
                        timestamp: new Date()
                    }
                );
            } catch (error) {
                console.log('Analytics save failed:', error);
            }
        }
    }
    
    setupEventListeners() {
        this.trackUserBehavior();
        
        // Track page visibility
        document.addEventListener('visibilitychange', () => {
            if (document.hidden) {
                this.trackEvent('page_hide');
            } else {
                this.trackEvent('page_show');
            }
        });
        
        // Track form interactions
        document.addEventListener('focusin', (e) => {
            if (e.target.matches('input, textarea, select')) {
                this.trackEvent('form_interaction', {
                    field_name: e.target.name || e.target.id,
                    field_type: e.target.type
                });
            }
        });
    }
}

// Initialize analytics
const analytics = new AdvancedAnalytics();
```

## 🎨 CSS Styles for New Features

```css
/* Add these styles to your existing CSS */

/* Modal Styles */
.modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2000;
    padding: 20px;
}

.modal-content {
    background: var(--white);
    border-radius: 15px;
    padding: 30px;
    max-width: 400px;
    width: 100%;
    position: relative;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.close {
    position: absolute;
    top: 15px;
    right: 20px;
    font-size: 24px;
    cursor: pointer;
    color: #999;
}

.close:hover {
    color: var(--primary-purple);
}

/* Auth Forms */
.auth-form h2 {
    text-align: center;
    margin-bottom: 20px;
    color: var(--primary-purple);
}

.auth-form .form-group {
    margin-bottom: 15px;
}

.auth-form label {
    display: block;
    margin-bottom: 5px;
    font-weight: 500;
    color: var(--dark);
}

.auth-form input {
    width: 100%;
    padding: 10px;
    border: 2px solid var(--primary-pink);
    border-radius: 8px;
    font-size: 14px;
    transition: border-color 0.3s;
}

.auth-form input:focus {
    outline: none;
    border-color: var(--primary-purple);
}

.auth-form .btn {
    width: 100%;
    margin-top: 10px;
}

.auth-form p {
    text-align: center;
    margin-top: 15px;
    font-size: 14px;
}

.auth-form a {
    color: var(--primary-purple);
    text-decoration: none;
}

.auth-form a:hover {
    text-decoration: underline;
}

/* Profile Dropdown */
.profile-dropdown {
    position: absolute;
    top: 100%;
    right: 0;
    background: var(--white);
    border-radius: 8px;
    box-shadow: var(--shadow);
    min-width: 180px;
    z-index: 1000;
}

.dropdown-content a {
    display: block;
    padding: 12px 16px;
    text-decoration: none;
    color: var(--dark);
    border-bottom: 1px solid #eee;
    transition: background-color 0.3s;
}

.dropdown-content a:hover {
    background: var(--light-purple);
    color: var(--primary-purple);
}

.dropdown-content a:last-child {
    border-bottom: none;
}

/* Cart Sidebar */
.cart-sidebar {
    position: fixed;
    top: 0;
    right: -400px;
    width: 400px;
    height: 100%;
    background: var(--white);
    box-shadow: -5px 0 15px rgba(0, 0, 0, 0.1);
    z-index: 1500;
    transition: right 0.3s ease;
    display: flex;
    flex-direction: column;
}

.cart-sidebar.active {
    right: 0;
}

.cart-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    z-index: 1400;
    opacity: 0;
    visibility: hidden;
    transition: all 0.3s ease;
}

.cart-overlay.active {
    opacity: 1;
    visibility: visible;
}

.cart-header {
    padding: 20px;
    border-bottom: 1px solid #eee;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.cart-header h3 {
    margin: 0;
    color: var(--primary-purple);
}

.close-btn {
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: #999;
}

.cart-body {
    flex: 1;
    overflow-y: auto;
    padding: 20px;
}

.sidebar-cart-item {
    display: flex;
    align-items: center;
    gap: 10px;
    padding: 10px 0;
    border-bottom: 1px solid #eee;
}

.item-thumb {
    width: 50px;
    height: 50px;
    object-fit: cover;
    border-radius: 5px;
}

.item-details {
    flex: 1;
}

.item-name {
    font-size: 14px;
    font-weight: 500;
    margin-bottom: 2px;
}

.item-price {
    font-size: 12px;
    color: var(--primary-purple);
}

.remove-btn {
    background: none;
    border: none;
    color: #999;
    cursor: pointer;
    font-size: 18px;
    padding: 5px;
}

.remove-btn:hover {
    color: #ff4444;
}

.cart-footer {
    padding: 20px;
    border-top: 1px solid #eee;
}

.cart-total {
    margin-bottom: 15px;
    text-align: center;
    font-size: 18px;
}

.cart-footer .btn {
    width: 100%;
    margin-bottom: 10px;
}

/* Advanced Wishlist */
.wishlist-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 30px;
    flex-wrap: wrap;
    gap: 15px;
}

.item-count {
    color: var(--primary-purple);
    font-weight: normal;
}

.wishlist-actions {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
}

.empty-wishlist {
    text-align: center;
    padding: 60px 20px;
}

.empty-icon {
    font-size: 80px;
    margin-bottom: 20px;
}

.wishlist-card {
    background: var(--white);
    border-radius: 12px;
    overflow: hidden;
    box-shadow: var(--shadow);
    transition: transform 0.3s, box-shadow 0.3s;
    position: relative;
}

.wishlist-card:hover {
    transform: translateY(-5px);
    box-shadow: 0 8px 25px rgba(0, 0, 0, 0.15);
}

.wishlist-card .item-image {
    position: relative;
    height: 200px;
    overflow: hidden;
}

.wishlist-card .item-image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
}

.item-overlay {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    align-items: center;
    justify-content: center;
    opacity: 0;
    transition: opacity 0.3s;
}

.wishlist-card:hover .item-overlay {
    opacity: 1;
}

.quick-view-btn {
    background: var(--white);
    border: none;
    border-radius: 50%;
    width: 40px;
    height: 40px;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    color: var(--primary-purple);
}

.wishlist-card .item-info {
    padding: 20px;
}

.wishlist-card .item-name {
    font-size: 16px;
    font-weight: 600;
    margin-bottom: 8px;
    color: var(--dark);
}

.wishlist-card .item-price {
    font-size: 18px;
    color: var(--primary-purple);
    font-weight: bold;
    margin-bottom: 10px;
}

.item-meta {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 15px;
    font-size: 12px;
}

.added-date {
    color: #666;
}

.priority {
    padding: 2px 8px;
    border-radius: 12px;
    font-weight: 500;
    text-transform: uppercase;
}

.priority-low {
    background: #e3f2fd;
    color: #1976d2;
}

.priority-medium {
    background: #fff3e0;
    color: #f57c00;
}

.priority-high {
    background: #ffebee;
    color: #d32f2f;
}

.wishlist-card .item-actions {
    display: flex;
    gap: 8px;
    flex-wrap: wrap;
}

.wishlist-card .btn {
    flex: 1;
    min-width: auto;
    padding: 8px 12px;
    font-size: 12px;
    border-radius: 6px;
}

.btn-outline {
    background: transparent;
    color: var(--primary-purple);
    border: 1px solid var(--primary-purple);
}

.btn-outline:hover {
    background: var(--primary-purple);
    color: var(--white);
}

/* Payment Methods */
.payment-method {
    display: flex;
    align-items: center;
    gap: 15px;
    padding: 15px;
    border: 2px solid #eee;
    border-radius: 10px;
    margin-bottom: 10px;
    cursor: pointer;
    transition: all 0.3s;
}

.payment-method:hover {
    border-color: var(--primary-pink);
    background: var(--light-purple);
}

.payment-method input[type="radio"]:checked + .method-info {
    color: var(--primary-purple);
}

.payment-method input[type="radio"]:checked ~ .method-total {
    color: var(--primary-purple);
    font-weight: bold;
}

.method-info {
    flex: 1;
}

.method-name {
    font-weight: 600;
    margin-bottom: 4px;
}

.method-description {
    font-size: 14px;
    color: #666;
}

.method-fee {
    font-size: 12px;
    color: #ff6b35;
    margin-top: 2px;
}

.method-total {
    font-size: 18px;
    font-weight: 600;
}

/* UPI Modal */
.upi-modal {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.8);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 2000;
    padding: 20px;
}

.upi-content {
    background: var(--white);
    border-radius: 15px;
    padding: 30px;
    max-width: 400px;
    width: 100%;
    text-align: center;
}

.upi-amount {
    font-size: 32px;
    font-weight: bold;
    color: var(--primary-purple);
    margin: 20px 0;
}

.upi-apps {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 15px;
    margin: 20px 0;
}

.upi-app {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
    padding: 15px;
    border: 1px solid #eee;
    border-radius: 10px;
    text-decoration: none;
    color: var(--dark);
    transition: all 0.3s;
}

.upi-app:hover {
    border-color: var(--primary-purple);
    background: var(--light-purple);
}

.upi-app img {
    width: 40px;
    height: 40px;
    border-radius: 8px;
}

.upi-qr {
    margin: 20px 0;
    padding: 20px;
    border: 1px dashed #ccc;
    border-radius: 10px;
}

/* Notifications */
.notification {
    position: fixed;
    top: 100px;
    right: 20px;
    padding: 15px 20px;
    border-radius: 8px;
    color: var(--white);
    font-weight: 500;
    z-index: 1000;
    transform: translateX(100%);
    transition: transform 0.3s ease;
    max-width: 300px;
}

.notification.show {
    transform: translateX(0);
}

.notification.success {
    background: #28a745;
}

.notification.error {
    background: #dc3545;
}

.notification.info {
    background: #17a2b8;
}

.notification.warning {
    background: #ffc107;
    color: var(--dark);
}

/* Responsive Design */
@media (max-width: 768px) {
    .modal-content {
        margin: 10px;
        padding: 20px;
    }
    
    .cart-sidebar {
        width: 100%;
        right: -100%;
    }
    
    .wishlist-header {
        flex-direction: column;
        align-items: stretch;
        text-align: center;
    }
    
    .wishlist-actions {
        justify-content: center;
    }
    
    .upi-apps {
        grid-template-columns: 1fr;
        gap: 10px;
    }
    
    .upi-app {
        flex-direction: row;
        justify-content: flex-start;
        text-align: left;
    }
    
    .payment-method {
        flex-direction: column;
        align-items: stretch;
        text-align: center;
    }
    
    .method-total {
        margin-top: 10px;
    }
}
```

This comprehensive feature enhancement guide provides everything needed to transform your basic gift store into a world-class ecommerce platform. Each feature is designed to be implemented incrementally, allowing you to build and test as you go while maintaining your existing functionality.

Remember to:
1. Test each feature thoroughly before moving to the next
2. Keep user experience at the center of all decisions
3. Monitor analytics to understand user behavior
4. Iterate based on customer feedback
5. Focus on mobile optimization since most users shop on mobile

Your personal touch combined with these professional features will create a unique competitive advantage in the gift market!