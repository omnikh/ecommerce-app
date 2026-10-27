## **The problem statement.**
A web based application where a customer can visit , browse the different listed products or search for products and then purchase a product and get it delivered.  Different sellers can reg ister themselves and list their products and its inventory and manage orders, and an admin who can manage sellers (optional).

## The Users of the product.
####  The customer
#### the seller
#### The admin - manages sellers and gets overall statistics (not in MVP)


## Use cases
what each user in the system can do:

### Customer :
**Account**

-  Register 
-  Login 
-  Logout 
-  View/edit profile 
-  Manage addresses 
**Shopping**

-  Browse products 
-  Search products 
-  View product details 
-  Add product to cart 
-  Update/remove cart items 
**Order**

-  Place order 
-  View orders (Order history)
-  Track order 
-  Cancel order 
-  Return item/order


### Seller:
**Account**

-  Register 
-  Login 
**Products**

-  View products 
**Listing**

-  Create listing for a product 
-  Update listing 
-  Update inventory 
-  Delete/deactivate listing 
**Orders**

-  View orders containing their listings 
-  Manage/fulfill orders


## The Business Workflow:
#### Customer Flow:
- [ ]  The app open up in a logged out mode, in which the user can browse products and rome around the app, when he clicks on buy /add to cart button, he is prompted to login/ register.
- [ ] the user can register himself with phone no, email id and Name in which the phone no and email id has to be unique fields.
- [ ] The user can add products to cart, the cart can have a max limit of 10 products with max limit of 10 qty on each product. he can check out the products from the cart. The use can directly go to a check out page by directly clicking on place order.
- [ ] between browsing the products and placing and order/adding to the cart, user can select a product to view its details. there is one important detail here, the default address shows the expected time of delivery automatically based on the pincode serviceability.
- [ ] At the check out page, the user can confirm the order in the 3 step process: Step 1. Address, Quantity, and Discount (not included in mvp) :  edit the default address or change the address from the list of available addresses, edit the qty available for each product in the check out page, add/remove any coupons for discount. Step 2: Payment type selection: In this step suer can select form the list of avaialble methods, his payment type and do the payment. After the payment confirmation, the order can finally be placed. if COD ..we have to check if cod is applicable for the product, if applicable, the order can be confirmed directly without any payment confirmation. While raising the qty of the product at the checkout page, the inventory has to be checked for availability of the product listing and should be denied raising the qty if not available.

#### Seller Flow:
- [ ] The user can register himself as a seller through the navigation bar - become a seller option.  the user can directly log in using his email/phone no. via otp or password. if it is a seller account, it will show the seller dashboard along with the default customer dashboard.
- [ ] the seller can, from the existing list of products, select a product and add a listing for it for this own inventory against it. 
- [ ] The seller can View all its product listings, deactivate or delete an existing listing, edit an existing listing for its quantity, or the product details, pricing, etc.
- [ ] The seller's dashboard has an order management section, in which he can see the new order 
- [ ] the initial state in which the seller recieves the order is the confirmed state. then he can accpet it or reject it on which it can turn into processing or rejected state. then when seller packs the order, he can turn it into ready to ship stage at which the application approaches the delivery management system to create a shipment. the further status is updated to the customer in his order history about the status of the application.
- [ ] Like the customer, seller can also edit his profile, update his details, change password, etc.


## Data modelling:
 

domain entities:

....

user

roles

user-roles-mapping

role-permissions-mapping

permissions

address

products

categories

Listings.

order

order_items

cart 

cart_items

shipment

shipment-order-items

shipment_event_tracking

order-address-snapshot















