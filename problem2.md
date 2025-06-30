1. Document Growth Impact
If a document keeps growing (e.g., frequently pushing into an array), MongoDB may need to relocate the document if it exceeds the original space allocated.
Risk:
    • Performance degradation
    • Increased memory fragmentation
    • Higher disk I/O due to document relocation

2. Storing File Metadata
Best Practice: Store only metadata (name, URL, type) in MongoDB and upload actual files (e.g., images, PDFs) to services like AWS S3.
Why?
    • MongoDB is not optimized for large binary storage.
    • Cloud storage provides better scalability, performance, and cost efficiency.

3. Large Collections with Millions of Documents
Best Practices:
    • Use proper indexing (on commonly queried fields)
    • Use compound indexes
    • Use sharding for horizontal scalability
    • Archive old/inactive data

4. Should You Use Mongoose Timestamps?
Usefulness: Automatically adds createdAt and updatedAt fields.
Enable when:
    • Tracking changes
    • Sorting by creation/update time
Disable when:
    • You need custom time tracking
    • Saving storage on small/embedded documents

5. Why Normalize in MongoDB?
Even in a NoSQL database:
Normalize when:
    • Data consistency is critical
    • Relationships are deeply nested
    • Updates are frequent across similar data

6. Email Uniqueness Enforcement
Set unique index in schema:
email: { type: String, unique: true }
Race Conditions: If two users register with the same email simultaneously, only one succeeds. Catch E11000 duplicate key error to handle this.

7. Handling API Rate Limits
Use middleware like express-rate-limit:
const rateLimit = require("express-rate-limit");
app.use(rateLimit({ windowMs: 1 * 60 * 1000, max: 100 }));
For distributed systems, use Redis-backed rate limiting.

8. Global Middleware vs Route-Level Middleware
Global: Use for authentication, logging, CORS.
Route-Level: Use for specific permission checks or validation on certain routes only.

9. Default Values in Schema
Benefits:
    • Ensures data consistency
    • Reduces backend validation code
Downsides:
    • May override user intent
    • Might hide logic bugs if defaults are incorrectly assumed

10. Why Use Virtual Populate Instead of Regular Populate?
Use Virtual Populate:
    • When referenced data is not stored in an array of IDs
    • To avoid denormalizing reference IDs in documents

11. Benefits of Lean Queries
.lean() returns plain JS objects instead of Mongoose documents.
When to use:
    • In read-heavy operations
    • When no virtuals, hooks, or setters are needed
Benefits:
    • Better performance
    • Lower memory usage

12. Should You Always Use .populate()?
Avoid populate when:
    • You need high performance
    • Populating leads to multiple expensive queries
Alternatives:
    • Embed required data
    • Use aggregation pipelines

13. Handling Expired Sessions or Tokens
JWT:
    • Use short-lived access tokens + refresh tokens
    • Store refresh tokens securely
    • On expiry, verify and issue new access token
Sessions:
    • Invalidate old sessions
    • Use expiration with secure cookie/session storage

14. Rate-Limited Aggregation Queries
Best Practices:
    • Index before aggregation
    • Use $match and $project early
    • Cache results when possible
    • Paginate long-running operations

15. Audit Trails in Schemas
Method 1: Embed logs in the same document:
auditLogs: [
  { changedBy: String, changes: Object, date: Date }
]
Method 2: Create a separate audit log collection:
{
  docType: "User",
  docId: ObjectId,
  operation: "UPDATE",
  changedBy: String,
  changes: Object,
  timestamp: Date
}
