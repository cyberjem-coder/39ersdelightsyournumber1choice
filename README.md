# 39ers Delights POS — Netlify-ready package

## What is preserved
- The uploaded POS HTML is retained as the application base.
- Existing local POS menu, cart, discounts/promotions, VAT, inventory, checkout, receipt, sales, audit and CEO reporting code is retained.
- The uploaded Google Sheets Apps Script and setup guide are included unchanged.
- Google Sheets continues to receive the existing POS reporting snapshots. Netlify is used only for the public mobile menu and mobile-order queue.

## Mobile order flow
1. The cashier opens the POS on the deployed Netlify site. The first POS browser publishes the menu and registers its device key in Netlify Blobs.
2. The QR code opens `/?customerOrder=1&station=...` and reads the public menu from `/api/mobile-menu`.
3. Customer submission creates a `PENDING` order at `/api/mobile-orders`.
4. The cashier's POS polls the protected pending queue every five seconds.
5. Confirming an order validates every menu item, variant and tracked stock quantity, then changes the order to `CONFIRMED` and imports the lines into the active POS cart. It does not create a sale or deduct stock.
6. Existing `processTransaction()` checkout remains the only route that records the sale, deducts stock and triggers the existing Google Sheets reporting sync.
7. Rejecting changes the order to `REJECTED`.


## Sold Out availability (added feature)

The POS keeps the existing inventory quantity and checkout workflow. A separate Sold Out setting adds manual control without changing menu prices, variants, promotions, VAT, sales, receipts, reports, or Google Sheets reporting.

- **Automatic** is the default. A tracked item is marked **SOLD OUT** when its inventory quantity reaches zero. When stock is replenished above zero, it becomes available again automatically unless it has a manual override.
- **Mark SOLD OUT** manually hides the item's add-to-cart/order buttons even when a positive stock quantity is recorded. Choose **Automatic** to remove the manual override and return to inventory-based availability. A zero-stock item remains sold out even in Automatic mode.
- The cashier POS shows a Sold Out badge and disables item choices. POS cart and completed-checkout checks remain active.
- The published mobile QR menu receives the availability state and quantity. Customer menu pages refresh the status periodically, check the latest published menu again at submission, and disable sold-out choices. The Netlify order function also checks the latest published snapshot.
- Mobile-order confirmation still imports items into the POS cart only. It does not deduct stock or record a sale. The normal cashier checkout remains the only operation that deducts inventory and records the sale.

**Important inventory scope:** The current project stores live inventory and the manual override in the cashier's browser profile (`localStorage`). Netlify receives a published snapshot for QR customers. This supports the existing single-cashier-browser setup; it is not a shared multi-cashier live inventory database. Pending mobile orders do not reserve stock. The cashier's import and checkout validations remain the final safeguards if multiple customers submit orders close together. Keep using the same cashier computer and Chrome profile, and retain a backup of the original project.

## Deploy
1. Extract this ZIP into a new Git repository and push it to GitHub, GitLab or another Git provider supported by Netlify. This project includes serverless Functions, so do not use a static-only drag-and-drop deploy.
2. In Netlify, import/connect that repository and deploy with the included `netlify.toml`. No build command is required; publish directory is `public`.
3. Confirm Netlify Blobs is available for the site (it is accessed by the Functions automatically).
4. Open the deployed site in the cashier POS browser first. Keep using that same browser/device profile: it stores the registered POS device key in localStorage.
5. Open the Customer QR / Mobile Order panel. Wait for the status to say the menu has been published to Netlify. Print or copy the QR link only after successful publication.
6. In the existing POS, configure Google Sheets Sync using the companion Apps Script URL as before. The mobile-order queue does not require the Apps Script URL.
7. Test with a low-value test order: submit from a phone, confirm on the POS, verify the items appear in the cart, and confirm that Sales Reports and Google Sheets do not gain a sale until normal checkout is completed.

## Important operational notes
- The first POS browser that successfully publishes the menu becomes the authorized cashier device. Do not clear its browser site data or move the POS to a different browser without planning a key reset/migration.
- The QR page and customer order submission are public by design. Do not collect payment card details or sensitive customer data in order notes.
- This package does not include your Netlify account, site URL, deployed state, or a live test. Deployment and end-to-end testing are still required.
- Keep a backup of the original uploaded POS HTML and Google Apps Script before replacing any live site files.

## Repair cashier-device authorization (only if you see "POS device not authorized")
1. In Netlify, open **Site configuration → Environment variables** (the wording may vary by dashboard version).
2. Add an environment variable named `POS_DEVICE_RESET_TOKEN`. Set its value to a new private random code at least 24 characters long. Do not share this code.
3. Trigger a new deploy so the Function receives the variable.
4. Open `https://YOUR-SITE-NAME.netlify.app/reset-pos-device.html` in your browser.
5. Enter the private reset code and click **Reset cashier authorization**. The page should confirm that authorization was reset.
6. On the intended cashier computer and Chrome profile, open the POS website, then open the Customer QR / Mobile Order panel. The browser should register its current device key and publish the menu.
7. Verify `https://YOUR-SITE-NAME.netlify.app/api/mobile-menu` now returns a non-empty `menu` array.
8. Once the cashier device is working, remove the `POS_DEVICE_RESET_TOKEN` environment variable from Netlify and redeploy. Keep the code private until it is removed.

This reset deletes only the stored cashier-device key hash from Netlify Blobs. It does not delete the customer menu or mobile order records.


## Customer QR mobile usability additions

This update adds customer-order-only interface improvements in `public/index.html` and includes `public/assets/39ers-delights-jingle.mp3`. Existing cashier POS screens and transactional workflows are not intentionally changed.

- Category buttons now re-render their active gold highlight on every selection.
- Menu cards already added to the tray receive a persistent visual highlight and an item count.
- Dine-In/Takeout and category selections provide a small tap sound using the browser Web Audio API. Sound feedback is optional and does not affect ordering.
- A sticky checkout bar is available at the top of the mobile menu and stays synchronized with the tray count and total; the existing Submit Order button remains in the tray. Both call the same order submission function.
- The supplied jingle is stored as a project asset and the page attempts to play it once on each fresh QR-menu visit. Mobile browsers (especially iPhone Safari) may block autoplay with sound until the visitor taps Play jingle. The visible Play/Pause control is the fallback. The track does not loop.

The jingle file is bundled in the deployable project, so it is not dependent on a temporary local upload. To update it in the future, replace `public/assets/39ers-delights-jingle.mp3` with the new audio file while keeping the same name, then deploy.

The changes still require deployment and real-device testing on iOS and Android. Browser autoplay rules, silent mode, and device volume remain outside the website's control.
