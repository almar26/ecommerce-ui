<template>

  <div class="shop-page">

    <!-- =========================================

         HEADER

    ========================================== -->

    <v-app-bar flat height="76" class="shop-header">

      <v-container class="d-flex align-center">

        <!-- LOGO -->

        <div class="d-flex align-center ga-2 logo-section">

          <v-icon icon="mdi-cart" color="primary" size="38" />



          <span class="text-h5 font-weight-bold">

            Shop<span class="text-primary">Kart</span>

          </span>

        </div>



        <!-- SEARCH -->

        <v-text-field

          v-model="search"

          placeholder="Search for products..."

          prepend-inner-icon="mdi-magnify"

          variant="outlined"

          density="compact"

          hide-details

          clearable

          class="search-box mx-8"

        />



        <v-spacer />



        <!-- MANAGE PRODUCTS -->

        <v-btn

          color="primary"

          variant="tonal"

          prepend-icon="mdi-package-variant"

          to="/products"

          class="mr-2 d-none d-md-flex"

        >

          Manage Products

        </v-btn>



        <!-- ACCOUNT -->

        <v-btn icon variant="text">

          <v-icon>mdi-account-outline</v-icon>



          <v-tooltip activator="parent"> Account </v-tooltip>

        </v-btn>



        <!-- WISHLIST -->

        <v-btn icon variant="text">

          <v-icon>mdi-heart-outline</v-icon>



          <v-tooltip activator="parent"> Wishlist </v-tooltip>

        </v-btn>



        <!-- CART -->

        <v-badge :content="cartCount" color="error" offset-x="8" offset-y="8">

          <v-btn icon variant="text" @click="cartDrawer = true">

            <v-icon> mdi-cart-outline </v-icon>

          </v-btn>

        </v-badge>

      </v-container>

    </v-app-bar>



    <!-- =========================================

         MOBILE SEARCH

    ========================================== -->

    <div class="d-md-none px-4 py-3 bg-white">

      <v-text-field

        v-model="search"

        placeholder="Search products..."

        prepend-inner-icon="mdi-magnify"

        variant="outlined"

        density="compact"

        hide-details

        clearable

      />



      <v-btn

        color="primary"

        variant="tonal"

        prepend-icon="mdi-package-variant"

        to="/products"

        block

        class="mt-3"

      >

        Manage Products

      </v-btn>

    </div>



    <!-- =========================================

         NAVIGATION

    ========================================== -->

    <div class="category-navigation">

      <v-container class="d-flex align-center ga-1 overflow-x-auto">

        <v-btn

          variant="text"

          :color="selectedCategory === 'All' ? 'primary' : undefined"

          @click="selectedCategory = 'All'"

        >

          All Categories

        </v-btn>



        <v-btn

          v-for="category in categories"

          :key="category"

          variant="text"

          :color="selectedCategory === category ? 'primary' : undefined"

          @click="selectedCategory = category"

        >

          {{ category }}

        </v-btn>

      </v-container>

    </div>



    <!-- =========================================

         MAIN CONTENT

    ========================================== -->

    <main>

      <v-container class="py-6">

        <!-- =====================================

             HERO SECTION

        ====================================== -->

        <v-card class="hero-card mb-10" rounded="xl" elevation="0">

          <v-row no-gutters>

            <v-col cols="12" md="6" class="d-flex align-center">

              <div class="hero-content">

                <div class="text-primary font-weight-bold mb-2">

                  NEW COLLECTION

                </div>



                <h1 class="hero-title">

                  Upgrade Your

                  <span class="text-primary"> Everyday </span>

                </h1>



                <p class="hero-description">

                  Discover great products at great prices. Shop electronics,

                  fashion, sports, home essentials and more.

                </p>



                <v-btn

                  color="primary"

                  size="large"

                  rounded="lg"

                  class="mt-3"

                  @click="scrollToProducts"

                >

                  Shop Now

                </v-btn>

              </div>

            </v-col>



            <v-col cols="12" md="6" class="hero-image-container">

              <div class="hero-art">

                <v-icon icon="mdi-headphones" size="150" color="primary" />



                <v-icon

                  icon="mdi-watch"

                  size="100"

                  color="black"

                  class="hero-watch"

                />



                <v-icon

                  icon="mdi-cellphone"

                  size="120"

                  color="primary"

                  class="hero-phone"

                />

              </div>

            </v-col>

          </v-row>

        </v-card>



        <!-- =====================================

             CATEGORY SECTION

        ====================================== -->

        <section class="mb-10">

          <div class="d-flex align-center justify-space-between mb-5">

            <div>

              <h2 class="text-h5 font-weight-bold">Shop by Category</h2>



              <p class="text-medium-emphasis mb-0">

                Find the products you need

              </p>

            </div>

          </div>



          <v-row>

            <v-col

              v-for="item in categoryCards"

              :key="item.name"

              cols="6"

              sm="4"

              md="3"

              lg="2"

            >

              <v-card

                class="category-card text-center pa-5"

                rounded="xl"

                elevation="0"

                @click="selectedCategory = item.name"

              >

                <v-avatar size="80" color="blue-lighten-5" class="mb-3">

                  <v-icon :icon="item.icon" color="primary" size="38" />

                </v-avatar>



                <div class="font-weight-bold">

                  {{ item.name }}

                </div>

              </v-card>

            </v-col>

          </v-row>

        </section>



        <!-- =====================================

             PRODUCTS

        ====================================== -->

        <section id="products" class="mb-10">

          <div

            class="d-flex flex-wrap align-center justify-space-between ga-3 mb-5"

          >

            <div>

              <h2 class="text-h5 font-weight-bold">Featured Products</h2>



              <p class="text-medium-emphasis mb-0">

                {{ filteredProducts.length }}

                product(s) available

              </p>

            </div>



            <v-btn

              color="primary"

              variant="text"

              prepend-icon="mdi-refresh"

              :loading="loading"

              @click="getProducts"

            >

              Refresh

            </v-btn>

          </div>



          <!-- LOADING -->

          <div v-if="loading" class="text-center py-16">

            <v-progress-circular indeterminate color="primary" size="55" />



            <p class="mt-4 text-medium-emphasis">Loading products...</p>

          </div>



          <!-- EMPTY -->

          <v-card

            v-else-if="filteredProducts.length === 0"

            rounded="xl"

            elevation="0"

            class="text-center pa-12"

          >

            <v-icon icon="mdi-package-variant-closed" size="70" color="grey" />



            <h3 class="mt-4">No Products Found</h3>



            <p class="text-medium-emphasis">Try another search or category.</p>



            <v-btn

              color="primary"

              variant="tonal"

              class="mt-3"

              @click="clearFilters"

            >

              Clear Filters

            </v-btn>

          </v-card>



          <!-- PRODUCT CARDS -->

          <v-row v-else>

            <v-col

              v-for="product in filteredProducts"

              :key="product.documentId"

              cols="12"

              sm="6"

              md="4"

              lg="3"

            >

              <v-card class="product-card h-100" rounded="xl" elevation="0">

                <!-- PRODUCT IMAGE -->

                <div class="product-image-container">

                  <v-img

                    v-if="product.image"

                    :src="getImageUrl(product.image)"

                    :alt="product.name"

                    height="240"

                    cover

                  />



                  <div v-else class="no-image">

                    <v-icon

                      icon="mdi-image-off-outline"

                      size="60"

                      color="grey-lighten-1"

                    />

                  </div>



                  <!-- WISHLIST -->

                  <v-btn

                    class="wishlist-btn"

                    icon

                    size="small"

                    elevation="1"

                    @click="toggleWishlist(product)"

                  >

                    <v-icon :color="isWishlist(product) ? 'error' : undefined">

                      {{

                        isWishlist(product) ? "mdi-heart" : "mdi-heart-outline"

                      }}

                    </v-icon>

                  </v-btn>



                  <!-- STATUS -->

                  <v-chip

                    v-if="product.stock === 0"

                    color="error"

                    size="small"

                    class="stock-label"

                  >

                    Out of Stock

                  </v-chip>

                </div>



                <!-- PRODUCT INFORMATION -->

                <v-card-text>

                  <div class="text-caption text-primary font-weight-bold mb-1">

                    {{ product.category || "Uncategorized" }}

                  </div>



                  <h3 class="product-name">

                    {{ product.name }}

                  </h3>



                  <p class="product-description">

                    {{ product.description || "No description available." }}

                  </p>



                  <div class="d-flex align-center justify-space-between mt-4">

                    <div>

                      <div class="text-h6 font-weight-bold text-primary">

                        {{ formatPrice(product.price) }}

                      </div>



                      <div class="text-caption text-medium-emphasis">

                        Stock:

                        {{ product.stock }}

                      </div>

                    </div>



                    <v-avatar

                      :color="getStockColor(product.stock)"

                      variant="tonal"

                      size="38"

                    >

                      <v-icon icon="mdi-package-variant" size="20" />

                    </v-avatar>

                  </div>

                </v-card-text>



                <v-card-actions class="px-4 pb-4">

                  <v-btn

                    color="primary"

                    variant="flat"

                    block

                    prepend-icon="mdi-cart-plus"

                    :disabled="product.stock <= 0 || !product.is_active"

                    @click="addToCart(product)"

                  >

                    {{ product.stock > 0 ? "Add to Cart" : "Out of Stock" }}

                  </v-btn>

                </v-card-actions>

              </v-card>

            </v-col>

          </v-row>

        </section>



        <!-- =====================================

             PROMOTIONS

        ====================================== -->

        <section class="mb-10">

          <v-row>

            <v-col cols="12" md="6">

              <v-card class="promo-card promo-green" rounded="xl" elevation="0">

                <div>

                  <div class="text-h5 font-weight-bold">Summer Sale</div>



                  <div class="text-h4 font-weight-black">Up to 50% OFF</div>



                  <v-btn

                    color="white"

                    variant="flat"

                    class="mt-3"

                    @click="scrollToProducts"

                  >

                    Shop Now

                  </v-btn>

                </div>



                <v-icon icon="mdi-palm-tree" size="100" class="promo-icon" />

              </v-card>

            </v-col>



            <v-col cols="12" md="6">

              <v-card

                class="promo-card promo-orange"

                rounded="xl"

                elevation="0"

              >

                <div>

                  <div class="text-h5 font-weight-bold">Top Brands</div>



                  <div class="text-h4 font-weight-black">Best Deals</div>



                  <v-btn

                    color="white"

                    variant="flat"

                    class="mt-3"

                    @click="scrollToProducts"

                  >

                    Shop Now

                  </v-btn>

                </div>



                <v-icon icon="mdi-shopping" size="100" class="promo-icon" />

              </v-card>

            </v-col>

          </v-row>

        </section>



        <!-- =====================================

             NEWSLETTER

        ====================================== -->

        <v-card rounded="xl" elevation="0" class="newsletter-card mb-10">

          <v-row align="center">

            <v-col cols="12" md="5">

              <div class="d-flex align-center ga-4">

                <v-avatar color="primary" variant="tonal" size="55">

                  <v-icon icon="mdi-email-outline" size="28" />

                </v-avatar>



                <div>

                  <h3 class="text-h6 font-weight-bold">Stay Updated</h3>



                  <p class="text-medium-emphasis mb-0">

                    Get the latest offers and new arrivals.

                  </p>

                </div>

              </div>

            </v-col>



            <v-col cols="12" md="7">

              <div class="d-flex ga-2">

                <v-text-field

                  v-model="email"

                  placeholder="Enter your email"

                  variant="outlined"

                  density="compact"

                  hide-details

                />



                <v-btn color="primary" height="40" @click="subscribe">

                  Subscribe

                </v-btn>

              </div>

            </v-col>

          </v-row>

        </v-card>

      </v-container>

    </main>



    <!-- =========================================

         FOOTER

    ========================================== -->

    <footer class="footer">

      <v-container class="py-10">

        <v-row>

          <v-col cols="12" md="4">

            <div class="d-flex align-center ga-2 mb-3">

              <v-icon icon="mdi-cart" color="white" size="32" />



              <span class="text-h5 font-weight-bold"> ShopKart </span>

            </div>



            <p class="footer-text">Shop smarter. Live better.</p>

          </v-col>



          <v-col cols="6" md="2">

            <div class="footer-title">Shop</div>



            <div class="footer-link">All Products</div>



            <div class="footer-link">New Arrivals</div>



            <div class="footer-link">Best Sellers</div>



            <div class="footer-link">Deals</div>

          </v-col>



          <v-col cols="6" md="2">

            <div class="footer-title">Help</div>



            <div class="footer-link">Track Order</div>



            <div class="footer-link">Returns</div>



            <div class="footer-link">Shipping</div>



            <div class="footer-link">FAQs</div>

          </v-col>



          <v-col cols="6" md="2">

            <div class="footer-title">About</div>



            <div class="footer-link">Our Story</div>



            <div class="footer-link">Careers</div>



            <div class="footer-link">Terms</div>



            <div class="footer-link">Privacy</div>

          </v-col>



          <v-col cols="6" md="2">

            <div class="footer-title">Admin</div>



            <v-btn

              color="primary"

              variant="tonal"

              prepend-icon="mdi-package-variant"

              to="/products"

            >

              Products

            </v-btn>

          </v-col>

        </v-row>



        <v-divider class="my-8" color="grey-darken-1" />



        <div class="text-center footer-text">

          © 2026 ShopKart. All rights reserved.

        </div>

      </v-container>

    </footer>



    <!-- =========================================

         CART DRAWER

    ========================================== -->

    <v-navigation-drawer
      v-model="cartDrawer"
      location="right"
      width="420"
      temporary
    >
      <!-- HEADER -->
      <v-list-item title="Your Cart" prepend-icon="mdi-cart">
        <template #append>
          <v-btn
            icon="mdi-close"
            variant="text"
            @click="cartDrawer = false"
          />
        </template>
      </v-list-item>

      <v-divider />

      <!-- EMPTY CART -->
      <div v-if="cart.length === 0" class="text-center pa-10">
        <v-icon icon="mdi-cart-outline" size="60" color="grey" />

        <h3 class="mt-4">Your cart is empty</h3>

        <p class="text-medium-emphasis mt-2">
          Add products before checking out.
        </p>
      </div>

      <!-- CART ITEMS -->
      <template v-else>
        <v-list>
          <v-list-item
            v-for="item in cart"
            :key="item.product.documentId"
            class="py-3"
          >
            <!-- PRODUCT IMAGE -->
            <template #prepend>
              <v-avatar rounded="lg" size="60" class="mr-3">
                <v-img
                  v-if="item.product.image"
                  :src="getImageUrl(item.product.image)"
                  :alt="item.product.name"
                  cover
                />

                <div
                  v-else
                  class="d-flex align-center justify-center bg-grey-lighten-3 w-100 h-100"
                >
                  <v-icon icon="mdi-image-off-outline" color="grey" />
                </div>
              </v-avatar>
            </template>

            <!-- PRODUCT NAME -->
            <v-list-item-title class="font-weight-bold">
              {{ item.product.name }}
            </v-list-item-title>

            <!-- PRICE / QUANTITY -->
            <v-list-item-subtitle>
              {{ formatPrice(item.product.price) }} × {{ item.quantity }}
            </v-list-item-subtitle>

            <!-- SUBTOTAL -->
            <div class="text-caption font-weight-bold text-primary mt-1">
              Subtotal: {{ formatPrice(getItemSubtotal(item)) }}
            </div>

            <!-- DELETE -->
            <template #append>
              <v-btn
                icon="mdi-delete-outline"
                color="error"
                variant="text"
                size="small"
                @click="removeFromCart(item.product)"
              />
            </template>
          </v-list-item>
        </v-list>

        <v-divider />

        <!-- CART TOTAL -->
        <div class="pa-5">
          <div class="d-flex justify-space-between mb-2">
            <span class="text-medium-emphasis">Total Items</span>

            <strong>{{ cartCount }}</strong>
          </div>

          <div class="d-flex justify-space-between align-center mb-5">
            <span class="text-subtitle-1 font-weight-bold">Total Amount</span>

            <span class="text-h6 font-weight-bold text-primary">
              {{ formatPrice(cartTotalAmount) }}
            </span>
          </div>

          <!-- CHECKOUT ALL -->
          <v-btn
            color="primary"
            prepend-icon="mdi-cart-check"
            size="large"
            block
            @click="openCheckout"
          >
            Checkout All
          </v-btn>
        </div>
      </template>
    </v-navigation-drawer>

    <!-- =========================================

         CHECKOUT ORDER SUMMARY

    ========================================== -->

    <v-dialog
      v-model="checkoutDialog"
      max-width="850"
      scrollable
    >
      <v-card rounded="xl">
        <!-- HEADER -->
        <v-card-title class="d-flex align-center justify-space-between pa-5">
          <div class="d-flex align-center ga-3">
            <v-avatar color="primary" variant="tonal">
              <v-icon>mdi-cart-check</v-icon>
            </v-avatar>

            <div>
              <div class="text-h5 font-weight-bold">Order Summary</div>

              <div class="text-caption text-medium-emphasis">
                Review all products before checkout
              </div>
            </div>
          </div>

          <v-btn
            icon="mdi-close"
            variant="text"
            @click="checkoutDialog = false"
          />
        </v-card-title>

        <v-divider />

        <!-- CONTENT -->
        <v-card-text class="pa-0">
          <!-- EMPTY -->
          <div v-if="cart.length === 0" class="text-center pa-12">
            <v-icon icon="mdi-cart-outline" size="70" color="grey" />

            <h3 class="mt-4">Your cart is empty</h3>
          </div>

          <!-- ORDER ITEMS -->
          <template v-else>
            <div class="order-summary-table">
              <v-table>
                <thead>
                  <tr>
                    <th>Product</th>
                    <th class="text-right">Unit Price</th>
                    <th class="text-center">Quantity</th>
                    <th class="text-right">Subtotal</th>
                  </tr>
                </thead>

                <tbody>
                  <tr
                    v-for="item in cart"
                    :key="item.product.documentId"
                  >
                    <!-- PRODUCT -->
                    <td>
                      <div class="d-flex align-center ga-3 py-2">
                        <v-avatar rounded="lg" size="65">
                          <v-img
                            v-if="item.product.image"
                            :src="getImageUrl(item.product.image)"
                            :alt="item.product.name"
                            cover
                          />

                          <div
                            v-else
                            class="d-flex align-center justify-center bg-grey-lighten-3 w-100 h-100"
                          >
                            <v-icon
                              icon="mdi-image-off-outline"
                              color="grey"
                            />
                          </div>
                        </v-avatar>

                        <div>
                          <div class="font-weight-bold">
                            {{ item.product.name }}
                          </div>

                          <div class="text-caption text-medium-emphasis">
                            {{ item.product.category || "Uncategorized" }}
                          </div>

                          <div class="text-caption text-medium-emphasis">
                            Available Stock: {{ item.product.stock }}
                          </div>
                        </div>
                      </div>
                    </td>

                    <!-- UNIT PRICE -->
                    <td class="text-right font-weight-medium">
                      {{ formatPrice(item.product.price) }}
                    </td>

                    <!-- QUANTITY -->
                    <td class="text-center">
                      <v-chip color="primary" variant="tonal">
                        {{ item.quantity }}
                      </v-chip>
                    </td>

                    <!-- SUBTOTAL -->
                    <td class="text-right font-weight-bold">
                      {{ formatPrice(getItemSubtotal(item)) }}
                    </td>
                  </tr>
                </tbody>
              </v-table>
            </div>

            <v-divider />

            <!-- TOTAL SUMMARY -->
            <div class="checkout-total-section">
              <div class="checkout-total-card">
                <div class="d-flex justify-space-between mb-3">
                  <span class="text-medium-emphasis">Total Items</span>

                  <span class="font-weight-bold">{{ cartCount }}</span>
                </div>

                <div class="d-flex justify-space-between mb-3">
                  <span class="text-medium-emphasis">Number of Products</span>

                  <span class="font-weight-bold">{{ cart.length }}</span>
                </div>

                <v-divider class="my-3" />

                <div class="d-flex justify-space-between align-center">
                  <span class="text-h6 font-weight-bold">Total Amount</span>

                  <span class="text-h5 font-weight-black text-primary">
                    {{ formatPrice(cartTotalAmount) }}
                  </span>
                </div>
              </div>
            </div>
          </template>
        </v-card-text>

        <v-divider />

        <!-- ACTIONS -->
        <v-card-actions class="pa-5">
          <v-btn
            variant="text"
            @click="checkoutDialog = false"
          >
            Back to Cart
          </v-btn>

          <v-spacer />

          <v-btn
            color="primary"
            prepend-icon="mdi-arrow-right"
            :disabled="cart.length === 0"
            @click="proceedFromOrderSummary"
          >
            Continue Checkout
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- =========================================

         ACTUAL CHECKOUT FORM

    ========================================== -->

    <v-dialog
      v-model="checkoutFormDialog"
      max-width="950"
      scrollable
      persistent
    >
      <v-card rounded="xl">
        <v-card-title class="d-flex align-center justify-space-between pa-5">
          <div class="d-flex align-center ga-3">
            <v-avatar color="primary" variant="tonal">
              <v-icon>mdi-credit-card-check-outline</v-icon>
            </v-avatar>

            <div>
              <div class="text-h5 font-weight-bold">Checkout</div>

              <div class="text-caption text-medium-emphasis">
                Enter customer details and choose a payment method
              </div>
            </div>
          </div>

          <v-btn
            icon="mdi-close"
            variant="text"
            :disabled="checkoutProcessing"
            @click="checkoutFormDialog = false"
          />
        </v-card-title>

        <v-divider />

        <v-card-text class="pa-6">
          <v-form
            ref="checkoutForm"
            @submit.prevent="confirmCheckout"
          >
            <v-row>
              <!-- CUSTOMER INFORMATION -->
              <v-col cols="12" md="7">
                <div class="text-subtitle-1 font-weight-bold mb-4">
                  Customer Information
                </div>

                <v-text-field
                  v-model="checkout.fullName"
                  label="Full Name"
                  prepend-inner-icon="mdi-account-outline"
                  variant="outlined"
                  :rules="[requiredRule]"
                  required
                />

                <v-text-field
                  v-model="checkout.contactNumber"
                  label="Contact Number"
                  prepend-inner-icon="mdi-phone-outline"
                  variant="outlined"
                  :rules="contactRules"
                  required
                />

                <v-text-field
                  v-model="checkout.email"
                  label="Email Address (Optional)"
                  prepend-inner-icon="mdi-email-outline"
                  variant="outlined"
                  :rules="emailRules"
                />

                <v-textarea
                  v-model="checkout.deliveryAddress"
                  label="Delivery Address"
                  prepend-inner-icon="mdi-map-marker-outline"
                  variant="outlined"
                  rows="3"
                  :rules="[requiredRule]"
                  required
                />

                <v-select
                  v-model="checkout.paymentMethod"
                  :items="paymentMethods"
                  label="Payment Method"
                  prepend-inner-icon="mdi-wallet-outline"
                  variant="outlined"
                  :rules="[requiredRule]"
                  required
                />

                <v-textarea
                  v-model="checkout.notes"
                  label="Order Notes (Optional)"
                  prepend-inner-icon="mdi-note-text-outline"
                  variant="outlined"
                  rows="2"
                />
              </v-col>

              <!-- CHECKOUT SUMMARY -->
              <v-col cols="12" md="5">
                <v-card
                  rounded="xl"
                  variant="tonal"
                  color="primary"
                  class="checkout-form-summary"
                >
                  <v-card-text>
                    <div class="text-h6 font-weight-bold mb-4">
                      Checkout Summary
                    </div>

                    <div
                      v-for="item in cart"
                      :key="item.product.documentId"
                      class="d-flex justify-space-between ga-3 mb-3"
                    >
                      <div class="text-body-2">
                        {{ item.product.name }}
                        <span class="text-medium-emphasis">
                          × {{ item.quantity }}
                        </span>
                      </div>

                      <div class="text-body-2 font-weight-bold text-no-wrap">
                        {{ formatPrice(getItemSubtotal(item)) }}
                      </div>
                    </div>

                    <v-divider class="my-4" />

                    <div class="d-flex justify-space-between mb-2">
                      <span>Total Items</span>
                      <strong>{{ cartCount }}</strong>
                    </div>

                    <div class="d-flex justify-space-between align-center">
                      <span class="text-subtitle-1 font-weight-bold">
                        Total Amount
                      </span>

                      <span class="text-h6 font-weight-black">
                        {{ formatPrice(cartTotalAmount) }}
                      </span>
                    </div>
                  </v-card-text>
                </v-card>

                <v-alert
                  type="info"
                  variant="tonal"
                  class="mt-4"
                  density="comfortable"
                >
                  This classroom checkout does not connect to a real payment
                  gateway. The selected payment method is recorded only for the
                  checkout demonstration.
                </v-alert>
              </v-col>
            </v-row>
          </v-form>
        </v-card-text>

        <v-divider />

        <v-card-actions class="pa-5">
          <v-btn
            variant="text"
            :disabled="checkoutProcessing"
            @click="backToOrderSummary"
          >
            Back to Order Summary
          </v-btn>

          <v-spacer />

          <v-btn
            color="primary"
            prepend-icon="mdi-check-circle-outline"
            size="large"
            :loading="checkoutProcessing"
            :disabled="checkoutProcessing || cart.length === 0"
            @click="confirmCheckout"
          >
            Place Order
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- =========================================

         CHECKOUT SUCCESS

    ========================================== -->

    <v-dialog
      v-model="checkoutSuccessDialog"
      max-width="520"
    >
      <v-card rounded="xl">
        <v-card-text class="text-center pa-8">
          <v-avatar
            color="success"
            variant="tonal"
            size="80"
            class="mb-4"
          >
            <v-icon
              icon="mdi-check-circle"
              size="48"
            />
          </v-avatar>

          <h2 class="text-h5 font-weight-bold">
            Order Successfully Placed!
          </h2>

          <p class="text-medium-emphasis mt-2 mb-6">
            Product stock has been updated in Strapi and your cart has been
            cleared.
          </p>

          <v-card
            v-if="completedOrder"
            variant="tonal"
            rounded="lg"
            class="text-left"
          >
            <v-card-text>
              <div class="d-flex justify-space-between mb-2">
                <span class="text-medium-emphasis">Reference</span>
                <strong>{{ completedOrder.reference }}</strong>
              </div>

              <div class="d-flex justify-space-between mb-2">
                <span class="text-medium-emphasis">Customer</span>
                <strong>{{ completedOrder.customer }}</strong>
              </div>

              <div class="d-flex justify-space-between mb-2">
                <span class="text-medium-emphasis">Payment</span>
                <strong>{{ completedOrder.paymentMethod }}</strong>
              </div>

              <div class="d-flex justify-space-between mb-2">
                <span class="text-medium-emphasis">Total Items</span>
                <strong>{{ completedOrder.totalItems }}</strong>
              </div>

              <v-divider class="my-3" />

              <div class="d-flex justify-space-between align-center">
                <span class="font-weight-bold">Total Amount</span>
                <span class="text-h6 font-weight-black text-success">
                  {{ formatPrice(completedOrder.totalAmount) }}
                </span>
              </div>
            </v-card-text>
          </v-card>
        </v-card-text>

        <v-card-actions class="pa-5 pt-0">
          <v-btn
            color="primary"
            block
            @click="checkoutSuccessDialog = false"
          >
            Continue Shopping
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- =========================================

         SNACKBAR

    ========================================== -->

    <v-snackbar

      v-model="snackbar.show"

      :color="snackbar.color"

      location="bottom right"

      timeout="2500"

    >

      {{ snackbar.message }}



      <template #actions>

        <v-btn variant="text" @click="snackbar.show = false"> Close </v-btn>

      </template>

    </v-snackbar>

  </div>

</template>



<script setup lang="ts">

// @ts-nocheck



definePageMeta({

  layout: false,

});



useHead({

  title: "ShopKart - E-Commerce",

});



const config = useRuntimeConfig();



const STRAPI_URL = config.public.strapiUrl;



// =============================================

// PRODUCTS

// =============================================



const products = ref([]);



const loading = ref(false);



const search = ref("");



const selectedCategory = ref("All");



// =============================================

// CART

// =============================================



const cart = ref([]);



const cartDrawer = ref(false);



// Controls the Order Summary dialog

const checkoutDialog = ref(false);

// Controls the customer/payment checkout form

const checkoutFormDialog = ref(false);

// Controls the successful checkout confirmation dialog

const checkoutSuccessDialog = ref(false);

// Vuetify form reference

const checkoutForm = ref(null);

// Shows loading state while validating and updating Strapi stock

const checkoutProcessing = ref(false);

// Keeps a small receipt-like summary after the cart is cleared

const completedOrder = ref(null);

// Customer and payment information

const checkout = reactive({

  fullName: "",

  contactNumber: "",

  email: "",

  deliveryAddress: "",

  paymentMethod: "",

  notes: "",

});

const paymentMethods = [

  "Cash on Delivery",

  "GCash",

  "Bank Transfer",

];

const requiredRule = (value) =>

  !!String(value ?? "").trim() || "This field is required";

const contactRules = [

  requiredRule,

  (value) =>

    /^[0-9+\- ()]{7,20}$/.test(String(value ?? "")) ||

    "Enter a valid contact number",

];

const emailRules = [

  (value) =>

    !value ||

    /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(String(value)) ||

    "Enter a valid email address",

];



const cartCount = computed(() => {

  return cart.value.reduce((total, item) => total + item.quantity, 0);

});



// =============================================

// CART TOTAL AMOUNT

// =============================================



const cartTotalAmount = computed(() => {

  return cart.value.reduce(

    (total, item) => {

      const price = Number(item.product.price || 0);

      const quantity = Number(item.quantity || 0);

      return total + price * quantity;

    },

    0,

  );

});



// =============================================

// WISHLIST

// =============================================



const wishlist = ref([]);



// =============================================

// NEWSLETTER

// =============================================



const email = ref("");



// =============================================

// CATEGORIES

// =============================================



const categories = [

  "Electronics",

  "Fashion",

  "Sports",

  "Bags",

  "Beauty",

  "Home & Living",

  "Toys",

];



const categoryCards = [

  {

    name: "Electronics",

    icon: "mdi-laptop",

  },

  {

    name: "Fashion",

    icon: "mdi-tshirt-crew",

  },

  {

    name: "Sports",

    icon: "mdi-basketball",

  },

  {

    name: "Bags",

    icon: "mdi-bag-personal",

  },

  {

    name: "Beauty",

    icon: "mdi-lipstick",

  },

  {

    name: "Home & Living",

    icon: "mdi-sofa",

  },

];



// =============================================

// FILTER PRODUCTS

// =============================================



const filteredProducts = computed(() => {

  let result = products.value.filter((product) => product.is_active !== false);



  if (selectedCategory.value !== "All") {

    result = result.filter(

      (product) => product.category === selectedCategory.value,

    );

  }



  if (search.value) {

    const keyword = search.value.toLowerCase().trim();



    result = result.filter((product) => {

      const name = product.name?.toLowerCase() || "";



      const description = product.description?.toLowerCase() || "";



      const category = product.category?.toLowerCase() || "";



      return (

        name.includes(keyword) ||

        description.includes(keyword) ||

        category.includes(keyword)

      );

    });

  }



  return result;

});



// =============================================

// READ PRODUCTS FROM STRAPI

// =============================================



async function getProducts() {

  loading.value = true;



  try {

    const response = await $fetch(`${STRAPI_URL}/api/products`, {

      query: {

        populate: "image",



        sort: "createdAt:desc",



        "pagination[pageSize]": 100,

      },

    });



    products.value = response.data || [];

  } catch (error) {

    console.error("Product loading error:", error);



    showSnackbar("Unable to load products from Strapi.", "error");

  } finally {

    loading.value = false;

  }

}



// =============================================

// PRODUCT IMAGE

// =============================================



function getImageUrl(image) {

  if (!image?.url) {

    return "";

  }



  if (image.url.startsWith("http://") || image.url.startsWith("https://")) {

    return image.url;

  }



  return STRAPI_URL + image.url;

}



// =============================================

// FORMAT PRICE

// =============================================



function formatPrice(price) {

  return new Intl.NumberFormat("en-PH", {

    style: "currency",

    currency: "PHP",

  }).format(Number(price || 0));

}



// =============================================

// CART

// =============================================



function addToCart(product) {

  if (product.stock <= 0) {

    showSnackbar("This product is out of stock.", "error");



    return;

  }



  const existingItem = cart.value.find(

    (item) => item.product.documentId === product.documentId,

  );



  if (existingItem) {

    if (existingItem.quantity >= product.stock) {

      showSnackbar("Maximum available stock reached.", "warning");



      return;

    }



    existingItem.quantity++;

  } else {

    cart.value.push({

      product,

      quantity: 1,

    });

  }



  showSnackbar(`${product.name} added to cart.`, "success");

}



function removeFromCart(product) {

  cart.value = cart.value.filter(

    (item) => item.product.documentId !== product.documentId,

  );

}



// =============================================

// ITEM SUBTOTAL

// =============================================



function getItemSubtotal(item) {

  const price = Number(item.product.price || 0);

  const quantity = Number(item.quantity || 0);

  return price * quantity;

}



// =============================================

// OPEN CHECKOUT

// =============================================



function openCheckout() {

  if (cart.value.length === 0) {

    showSnackbar("Your cart is empty.", "warning");

    return;

  }



  cartDrawer.value = false;

  checkoutDialog.value = true;

}



// =============================================

// CONTINUE CHECKOUT

// =============================================



function proceedFromOrderSummary() {

  if (cart.value.length === 0) {

    showSnackbar("Your cart is empty.", "warning");

    return;

  }



  checkoutDialog.value = false;

  checkoutFormDialog.value = true;

}



// =============================================

// BACK TO ORDER SUMMARY

// =============================================



function backToOrderSummary() {

  if (checkoutProcessing.value) {

    return;

  }



  checkoutFormDialog.value = false;

  checkoutDialog.value = true;

}



// =============================================

// RESET CHECKOUT FORM

// =============================================



function resetCheckoutForm() {

  checkout.fullName = "";

  checkout.contactNumber = "";

  checkout.email = "";

  checkout.deliveryAddress = "";

  checkout.paymentMethod = "";

  checkout.notes = "";



  checkoutForm.value?.resetValidation?.();

}



// =============================================

// CONFIRM ACTUAL CHECKOUT

// =============================================



async function confirmCheckout() {

  if (cart.value.length === 0) {

    showSnackbar("Your cart is empty.", "warning");

    return;

  }



  const validation = await checkoutForm.value?.validate();



  if (validation && !validation.valid) {

    showSnackbar("Please complete the required checkout fields.", "warning");

    return;

  }



  checkoutProcessing.value = true;



  try {

    /*
      STEP 1:
      Re-read every product from Strapi before changing stock.
      This prevents checkout from using an old stock value that
      may have changed after the product was added to the cart.
    */

    const validatedItems = [];



    for (const cartItem of cart.value) {

      const response = await $fetch(

        `${STRAPI_URL}/api/products/${cartItem.product.documentId}`,

        {

          query: {

            populate: "image",

          },

        },

      );



      const latestProduct = response?.data;



      if (!latestProduct) {

        throw new Error(

          `${cartItem.product.name} could not be found in Strapi.`,

        );

      }



      if (latestProduct.is_active === false) {

        throw new Error(

          `${latestProduct.name} is no longer available for purchase.`,

        );

      }



      const latestStock = Number(latestProduct.stock || 0);

      const requestedQuantity = Number(cartItem.quantity || 0);



      if (requestedQuantity <= 0) {

        throw new Error(`Invalid quantity for ${latestProduct.name}.`);

      }



      if (requestedQuantity > latestStock) {

        throw new Error(

          `Insufficient stock for ${latestProduct.name}. Available stock: ${latestStock}.`,

        );

      }



      validatedItems.push({

        documentId: latestProduct.documentId,

        name: latestProduct.name,

        price: Number(latestProduct.price || 0),

        quantity: requestedQuantity,

        currentStock: latestStock,

        newStock: latestStock - requestedQuantity,

      });

    }



    /*
      STEP 2:
      Update the stock of every validated product in Strapi.
    */

    for (const item of validatedItems) {

      await $fetch(

        `${STRAPI_URL}/api/products/${item.documentId}`,

        {

          method: "PUT",

          body: {

            data: {

              stock: item.newStock,

            },

          },

        },

      );

    }



    /*
      STEP 3:
      Create a small receipt summary before clearing the cart.
      This receipt is only stored in the current browser session.
    */

    const finalTotalItems = validatedItems.reduce(

      (total, item) => total + item.quantity,

      0,

    );



    const finalTotalAmount = validatedItems.reduce(

      (total, item) => total + item.price * item.quantity,

      0,

    );



    completedOrder.value = {

      reference: `CHK-${Date.now().toString().slice(-8)}`,

      customer: checkout.fullName,

      paymentMethod: checkout.paymentMethod,

      totalItems: finalTotalItems,

      totalAmount: finalTotalAmount,

    };



    /*
      STEP 4:
      Clear the cart only after all Strapi stock updates succeed.
    */

    cart.value = [];



    checkoutFormDialog.value = false;

    checkoutDialog.value = false;

    cartDrawer.value = false;



    resetCheckoutForm();



    /*
      STEP 5:
      Refresh products so the new stock values are immediately
      visible on the E-Commerce Homepage.
    */

    await getProducts();



    checkoutSuccessDialog.value = true;



    showSnackbar("Checkout completed successfully.", "success");

  } catch (error) {

    console.error("Checkout error:", error);



    showSnackbar(

      error?.data?.error?.message ||

        error?.data?.message ||

        error?.message ||

        "Checkout failed. Please try again.",

      "error",

    );

  } finally {

    checkoutProcessing.value = false;

  }

}



// =============================================

// WISHLIST

// =============================================



function toggleWishlist(product) {

  const index = wishlist.value.findIndex((id) => id === product.documentId);



  if (index >= 0) {

    wishlist.value.splice(index, 1);



    showSnackbar(`${product.name} removed from wishlist.`, "info");

  } else {

    wishlist.value.push(product.documentId);



    showSnackbar(`${product.name} added to wishlist.`, "success");

  }

}



function isWishlist(product) {

  return wishlist.value.includes(product.documentId);

}



// =============================================

// STOCK COLOR

// =============================================



function getStockColor(stock) {

  if (stock === 0) {

    return "error";

  }



  if (stock <= 5) {

    return "warning";

  }



  return "success";

}



// =============================================

// CLEAR FILTERS

// =============================================



function clearFilters() {

  search.value = "";



  selectedCategory.value = "All";

}



// =============================================

// SCROLL TO PRODUCTS

// =============================================



function scrollToProducts() {

  document.getElementById("products")?.scrollIntoView({

    behavior: "smooth",

  });

}



// =============================================

// NEWSLETTER

// =============================================



function subscribe() {

  if (!email.value) {

    showSnackbar("Please enter your email.", "warning");



    return;

  }



  showSnackbar("Thank you for subscribing!", "success");



  email.value = "";

}



// =============================================

// SNACKBAR

// =============================================



const snackbar = reactive({

  show: false,

  message: "",

  color: "success",

});



function showSnackbar(message, color = "success") {

  snackbar.message = message;



  snackbar.color = color;



  snackbar.show = true;

}



// =============================================

// INITIAL LOAD

// =============================================



onMounted(() => {

  getProducts();

});

</script>



<style scoped>

/* =============================================

   PAGE

============================================= */



.shop-page {

  min-height: 100vh;

  background: #f7f9fc;

}



/* =============================================

   HEADER

============================================= */



.shop-header {

  border-bottom: 1px solid rgba(0, 0, 0, 0.08);

}



.logo-section {

  white-space: nowrap;

}



.search-box {

  max-width: 520px;

}



/* =============================================

   NAVIGATION

============================================= */



.category-navigation {

  background: white;



  border-bottom: 1px solid rgba(0, 0, 0, 0.08);

}



/* =============================================

   HERO

============================================= */



.hero-card {

  overflow: hidden;



  background: linear-gradient(135deg, #e9f4ff 0%, #d9edff 100%);

}



.hero-content {

  padding: 65px 55px;

}



.hero-title {

  max-width: 500px;



  font-size: clamp(2.2rem, 5vw, 4.2rem);



  line-height: 1;



  font-weight: 900;

}



.hero-description {

  max-width: 480px;



  margin-top: 20px;



  font-size: 1.05rem;



  color: #5d6875;

}



.hero-image-container {

  min-height: 400px;



  display: flex;



  align-items: center;



  justify-content: center;

}



.hero-art {

  position: relative;



  width: 400px;



  height: 300px;



  display: flex;



  align-items: center;



  justify-content: center;

}



.hero-watch {

  position: absolute;



  right: 70px;



  bottom: 50px;

}



.hero-phone {

  position: absolute;



  right: 5px;



  top: 65px;



  transform: rotate(8deg);

}



/* =============================================

   CATEGORY

============================================= */



.category-card {

  cursor: pointer;



  background: white;



  border: 1px solid rgba(0, 0, 0, 0.06);



  transition:

    transform 0.2s ease,

    box-shadow 0.2s ease;

}



.category-card:hover {

  transform: translateY(-5px);



  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.08);

}



/* =============================================

   PRODUCT CARD

============================================= */



.product-card {

  overflow: hidden;



  background: white;



  border: 1px solid rgba(0, 0, 0, 0.08);



  transition:

    transform 0.2s ease,

    box-shadow 0.2s ease;

}



.product-card:hover {

  transform: translateY(-5px);



  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.1);

}



.product-image-container {

  position: relative;



  background: #f8f9fa;

}



.no-image {

  height: 240px;



  display: flex;



  align-items: center;



  justify-content: center;

}



.wishlist-btn {

  position: absolute;



  top: 12px;



  right: 12px;



  background: white;

}



.stock-label {

  position: absolute;



  left: 12px;



  top: 12px;

}



.product-name {

  min-height: 48px;



  font-size: 1.05rem;



  font-weight: 700;

}



.product-description {

  height: 42px;



  margin-top: 8px;



  overflow: hidden;



  display: -webkit-box;



  -webkit-line-clamp: 2;



  -webkit-box-orient: vertical;



  color: #737b84;



  font-size: 0.9rem;

}



/* =============================================

   PROMOTIONS

============================================= */



.promo-card {

  min-height: 190px;



  position: relative;



  overflow: hidden;



  padding: 35px;



  display: flex;



  align-items: center;



  justify-content: space-between;

}



.promo-green {

  background: linear-gradient(135deg, #b8f3d6, #55d59d);

}



.promo-orange {

  background: linear-gradient(135deg, #ffdfc3, #ffaf76);

}



.promo-icon {

  opacity: 0.65;

}



/* =============================================

   NEWSLETTER

============================================= */



.newsletter-card {

  padding: 28px;



  background: #eaf4ff;

}



/* =============================================

   FOOTER

============================================= */



.footer {

  background: #0d2036;



  color: white;

}



.footer-title {

  margin-bottom: 15px;



  font-weight: 700;

}



.footer-link {

  margin-bottom: 9px;



  color: #bbc5d0;



  font-size: 0.9rem;



  cursor: pointer;

}



.footer-link:hover {

  color: white;

}



.footer-text {

  color: #bbc5d0;

}



/* =============================================

   ORDER SUMMARY

============================================= */



.order-summary-table {

  overflow-x: auto;

}



.checkout-total-section {

  display: flex;

  justify-content: flex-end;

  padding: 24px;

}



.checkout-total-card {

  width: 100%;

  max-width: 400px;

  padding: 22px;

  border-radius: 16px;

  background: #f7f9fc;

  border: 1px solid rgba(0, 0, 0, 0.08);

}



/* =============================================

   CHECKOUT FORM

============================================= */



.checkout-form-summary {

  position: sticky;

  top: 0;

}



/* =============================================

   RESPONSIVE

============================================= */



@media (max-width: 960px) {

  .hero-content {

    padding: 45px 30px;

  }



  .hero-image-container {

    min-height: 280px;

  }



  .hero-art {

    transform: scale(0.75);

  }

}



@media (max-width: 600px) {

  .hero-content {

    padding: 40px 25px;

  }



  .hero-title {

    font-size: 2.6rem;

  }



  .hero-image-container {

    display: none;

  }



  .promo-card {

    min-height: 165px;



    padding: 25px;

  }



  .promo-icon {

    font-size: 70px !important;

  }



  .checkout-total-section {

    padding: 16px;

  }



  .checkout-total-card {

    max-width: 100%;

  }

}

</style>
