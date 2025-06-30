Mongoose Relationship & Middleware Examples
This document contains code implementations for common use-cases in MongoDB using Mongoose and Node.js.

1. Customer - Orders (One-to-Many)
Schemas:
// Customer.js
const mongoose = require("mongoose");
const customerSchema = new mongoose.Schema({ name: String, email: String });
module.exports = mongoose.model("Customer", customerSchema);
// Order.js
const mongoose = require("mongoose");
const orderSchema = new mongoose.Schema({
  customer: { type: mongoose.Schema.Types.ObjectId, ref: "Customer" },
  product: String,
  amount: Number,
});
module.exports = mongoose.model("Order", orderSchema);
Query:
const Order = require("./Order");
const getCustomerWithOrders = async (customerId) => {
  const customerOrders = await Order.find({ customer: customerId }).populate("customer");
  console.log(customerOrders);
};

2. Driver - Vehicle (One-to-One)
Schemas:
// Driver.js
const driverSchema = new mongoose.Schema({ name: String, licenseNumber: String });
module.exports = mongoose.model("Driver", driverSchema);
// Vehicle.js
const vehicleSchema = new mongoose.Schema({
  model: String,
  plateNumber: String,
  driver: { type: mongoose.Schema.Types.ObjectId, ref: "Driver", unique: true },
});
module.exports = mongoose.model("Vehicle", vehicleSchema);
Query:
const Vehicle = require("./Vehicle");
const getVehicleWithDriver = async (vehicleId) => {
  const vehicle = await Vehicle.findById(vehicleId).populate("driver");
  console.log(vehicle);
};

3. User - Roles (Many-to-Many via Array)
Schema:
// User.js
const userSchema = new mongoose.Schema({
  username: String,
  roles: [String],
});
module.exports = mongoose.model("User", userSchema);
Query:
const User = require("./User");
const getAdmins = async () => {
  const admins = await User.find({ roles: "admin" });
  console.log(admins);
};

4. Books by Category Count
Aggregation:
const Book = require("./Book");
const getBookCountByCategory = async () => {
  const result = await Book.aggregate([
    { $group: { _id: "$category", count: { $sum: 1 } } }
  ]);
  console.log(result);
};

5. Revenue per Product
Schema Snippet:
const orderSchema = new mongoose.Schema({
  products: [
    {
      name: String,
      price: Number,
      quantity: Number,
    },
  ],
});
Aggregation:
const Order = require("./Order");
const getRevenuePerProduct = async () => {
  const result = await Order.aggregate([
    { $unwind: "$products" },
    {
      $group: {
        _id: "$products.name",
        totalRevenue: { $sum: { $multiply: ["$products.price", "$products.quantity"] } },
      },
    },
  ]);
  console.log(result);
};

6. Average Rating per User
Schema Snippet:
const reviewSchema = new mongoose.Schema({
  user: { type: mongoose.Schema.Types.ObjectId, ref: "User" },
  rating: Number,
});
Aggregation:
const Review = require("./Review");
const getAverageRatingPerUser = async () => {
  const result = await Review.aggregate([
    { $group: { _id: "$user", avgRating: { $avg: "$rating" } } }
  ]);
  console.log(result);
};

7. JWT Auth Middleware
Middleware:
const jwt = require("jsonwebtoken");

const auth = (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ message: "Token missing" });

  try {
    const decoded = jwt.verify(token, "SECRET_KEY");
    req.userId = decoded.userId;
    next();
  } catch (err) {
    return res.status(403).json({ message: "Invalid token" });
  }
};

module.exports = auth;

8. Role-Based Access Control Middleware
Middleware:
const roleCheck = (requiredRole) => {
  return (req, res, next) => {
    const userRoles = req.userRoles || [];
    if (!userRoles.includes(requiredRole)) {
      return res.status(403).json({ message: "Access denied" });
    }
    next();
  };
};

module.exports = roleCheck;
Example Usage:
app.get("/admin", auth, roleCheck("admin"), (req, res) => {
  res.send("Welcome Admin");
});

Let me know if you want these bundled in a boilerplate folder or with Postman collection for testing.