# Canteen-Preorder-System
It's a preorder and takeaway system for our canteen.
canteen-checkout/
├─ src/
│  ├─ app.js
│  ├─ server.js
│  ├─ config/
│  │  └─ db.js
│  ├─ models/
│  │  └─ Order.js
│  ├─ routes/
│  │  └─ checkout.js
│  ├─ controllers/
│  │  └─ checkoutController.js
│  ├─ services/
│  │  ├─ orderService.js
│  │  └─ paymentService.js
│  ├─ paymentProviders/
│  │  ├─ mockProvider.js
│  │  └─ razorpayProvider.js   <-- skeleton
│  ├─ utils/
│  │  └─ idempotency.js
│  └─ middlewares/
│     └─ errorHandler.js
├─ tests/
│  ├─ orderService.test.js
│  └─ checkoutController.test.js
├─ .env.example
├─ package.json
└─ jest.config.js
{
  "name": "canteen-checkout",
  "version": "1.0.0",
  "main": "src/server.js",
  "scripts": {
    "start": "node src/server.js",
    "dev": "nodemon src/server.js",
    "test": "jest --runInBand"
  },
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.0.0",
    "dotenv": "^16.0.0",
    "body-parser": "^1.20.2",
    "crypto": "^1.0.1"
  },
  "devDependencies": {
    "jest": "^29.0.0",
    "supertest": "^6.3.0",
    "sinon": "^15.0.0"
  }
}
PORT=3000
MONGODB_URI=mongodb://localhost:27017/canteen
RAZORPAY_KEY_ID=your_key_id
RAZORPAY_KEY_SECRET=your_key_secret
PAYMENT_PROVIDER=mock   # or razorpay
const mongoose = require('mongoose');

async function connect(uri) {
  await mongoose.connect(uri, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });
  console.log('MongoDB connected');
}

module.exports = { connect };
const mongoose = require('mongoose');

const ItemSchema = new mongoose.Schema({
  itemId: String,
  name: String,
  quantity: { type: Number, required: true },
  price: { type: Number, required: true } // price in smallest currency unit (e.g., paise)
});

const OrderSchema = new mongoose.Schema({
  studentId: { type: String, required: true },
  items: [ItemSchema],
  totalAmount: { type: Number, required: true },
  currency: { type: String, default: 'INR' },
  status: { type: String, enum: ['created','paid','failed','cancelled'], default: 'created' },
  paymentProvider: { type: String, default: 'mock' },
  providerPaymentId: { type: String }, // id from gateway
  metadata: { type: Object },
  idempotencyKey: { type: String, index: true, sparse: true },
}, { timestamps: true });

module.exports = mongoose.model('Order', OrderSchema);
