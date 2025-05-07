# SaaS Feedback System

A GraphQL-based web service for collecting and managing user feedback on SaaS products, with web scraping capabilities using Puppeteer.

## Setup
1. Install Node.js (v16 or higher).
2. Clone the repository.
3. Run `npm install` to install dependencies.
4. Start the server with `npm start`.

## Usage
- Access the GraphQL playground at `http://localhost:4000`.
- Use queries and mutations to manage users, products, feedback, scraped reviews, and scraped products.
- For web scraping:
  - `scrapeReviews`: Provide a URL with review data structured with `.review`, `.rating`, and `.comment` selectors.
  - `scrapeProducts`: Provide a URL with product data; uses heuristics and Sephora-specific selectors (`.ProductTile-content`, `.css-1ma869u`, `.css-1f35s9q`).
- For feedback:
  - `createUser`: Create a user to submit feedback.
  - `createProduct`: Create products to link feedback (matches scraped products).
  - `submitFeedback`: Submit feedback with `userId`, `productId`, `rating`, and `comment`. Ensure `userId` and `productId` exist.
  - `feedbacks`: Query feedback for a `productId`.

## Testing Web Scraping
1. Save `dior-product-list.html` and `dior-product-reviews.html` in the project directory.
2. Serve them locally:
   ```bash
   python -m http.server 8000
   ```
3. Test `scrapeProducts` with `http://localhost:8000/dior-product-list.html`.
4. Test `scrapeReviews` with `http://localhost:8000/dior-product-reviews.html`.
5. For Sephora (`https://www.sephora.com/brand/dior/all`), ensure compliance with `robots.txt` and check console logs.

## Testing Feedback
1. Create a user:
   ```graphql
   mutation {
     createUser(username: "testUser", email: "test@example.com") {
       id
     }
   }
   ```
2. Create products matching scraped data:
   ```graphql
   mutation {
     createProduct(name: "Dior Addict Lip Glow Balm", description: "15 Colors") {
       id
     }
   }
   ```
3. Submit feedback with valid `userId` and `productId`:
   ```graphql
   mutation {
     submitFeedback(userId: "<user-id>", productId: "<product-id>", rating: 5, comment: "Great product!") {
       id
       rating
       comment
     }
   }
   ```
4. Query feedback:
   ```graphql
   query {
     feedbacks(productId: "<product-id>") {
       id
       rating
       comment
       user { username }
       product { name }
     }
   }
   ```
5. **Note**: Ensure `userId` and `productId` exist. Run `createUser` and `createProduct` first, and use the returned IDs. Invalid IDs will cause errors like "User with ID ... not found".

## Automated Scraping
- A daily cron job runs at midnight to scrape products from a configured URL (default: `http://localhost:8000/dior-product-list.html`).
- Update the `sourceUrl` in the cron job for production (e.g., `https://www.sephora.com/brand/dior/all`).
- Handles navigation timeouts with a 60-second limit and falls back to `domcontentloaded`.

## Dependencies
- `@apollo/server`: GraphQL server.
- `graphql`: GraphQL schema utilities.
- `puppeteer`: Web scraping (uses new headless mode).
- `cheerio`: HTML parsing.
- `uuid`: Unique ID generation.
- `node-cron`: Scheduling.

## Notes
- Data is stored in memory. For production, use MongoDB.
- Scraping uses Puppeteer’s new headless mode (`headless: "new"`).
- Includes Sephora-specific selectors and debug logs.
- Ensure compliance with `robots.txt` and terms of service.
- Use proxies or ScrapingBee for anti-scraping measures.