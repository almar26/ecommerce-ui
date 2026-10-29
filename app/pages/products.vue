<template>
  <v-layout class="product-layout">
    <!-- =========================================
         APP BAR
    ========================================== -->
    <v-app-bar flat height="76" class="shop-header">
      <v-container class="d-flex align-center">
        <!-- LOGO -->
        <NuxtLink to="/" class="logo-link d-flex align-center ga-2">
          <v-icon icon="mdi-cart" color="primary" size="38" />

          <span class="text-h5 font-weight-bold text-black">
            Shop<span class="text-primary">Kart</span>
          </span>
        </NuxtLink>

        <v-spacer />

        <!-- HOME -->
        <v-btn
          variant="text"
          prepend-icon="mdi-home-outline"
          to="/"
          class="mr-2"
        >
          Home
        </v-btn>

        <!-- CURRENT PAGE -->
        <v-btn
          color="primary"
          variant="tonal"
          prepend-icon="mdi-package-variant"
        >
          Manage Products
        </v-btn>
      </v-container>
    </v-app-bar>

    <!-- =========================================
         MAIN CONTENT
    ========================================== -->
    <v-main class="product-page">
      <v-container class="py-8">
        <!-- ========================================
             PAGE HEADER
        ========================================= -->
        <div
          class="d-flex flex-wrap align-center justify-space-between ga-4 mb-6"
        >
          <div>
            <h1 class="text-h4 font-weight-bold">Product Management</h1>

            <p class="text-medium-emphasis mt-1 mb-0">
              Manage your e-commerce products
            </p>
          </div>

          <!-- ADD PRODUCT -->
          <v-btn
            color="primary"
            prepend-icon="mdi-plus"
            size="large"
            @click="openCreateDialog"
          >
            Add Product
          </v-btn>
        </div>

        <!-- ========================================
             SEARCH
        ========================================= -->
        <v-card class="mb-6" rounded="lg" elevation="1">
          <v-card-text>
            <v-text-field
              v-model="search"
              label="Search product"
              prepend-inner-icon="mdi-magnify"
              variant="outlined"
              density="comfortable"
              clearable
              hide-details
            />
          </v-card-text>
        </v-card>

        <!-- ========================================
             LOADING
        ========================================= -->
        <div v-if="loading" class="text-center py-16">
          <v-progress-circular indeterminate color="primary" size="50" />

          <div class="mt-4 text-medium-emphasis">Loading products...</div>
        </div>

        <!-- ========================================
             EMPTY
        ========================================= -->
        <v-card
          v-else-if="filteredProducts.length === 0"
          rounded="lg"
          elevation="1"
        >
          <v-card-text class="text-center py-16">
            <v-icon icon="mdi-package-variant" size="64" color="grey" />

            <h3 class="mt-4">No Products Found</h3>

            <p class="text-medium-emphasis mt-2">
              Add your first product to get started.
            </p>
          </v-card-text>
        </v-card>

        <!-- ========================================
             PRODUCT TABLE
        ========================================= -->
        <v-card v-else rounded="lg" elevation="1" class="product-table-card">
          <v-table>
            <thead>
              <tr>
                <th>Image</th>
                <th>Name</th>
                <th>Category</th>
                <th>Price</th>
                <th>Stock</th>
                <th>Status</th>

                <th class="text-center">Actions</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="product in filteredProducts" :key="product.documentId">
                <!-- IMAGE -->
                <td>
                  <v-avatar rounded="lg" size="70" class="my-2">
                    <v-img
                      v-if="product.image"
                      :src="getImageUrl(product.image)"
                      :alt="product.name"
                      cover
                    />

                    <div
                      v-else
                      class="d-flex align-center justify-center bg-grey-lighten-3 w-100 h-100"
                    >
                      <v-icon icon="mdi-image-off" color="grey" />
                    </div>
                  </v-avatar>
                </td>

                <!-- NAME -->
                <td>
                  <div class="font-weight-bold">
                    {{ product.name }}
                  </div>

                  <div class="description-text text-medium-emphasis">
                    {{ product.description || "No description" }}
                  </div>
                </td>

                <!-- CATEGORY -->
                <td>
                  <v-chip color="primary" variant="tonal" size="small">
                    {{ product.category || "Uncategorized" }}
                  </v-chip>
                </td>

                <!-- PRICE -->
                <td class="font-weight-bold">
                  {{ formatPrice(product.price) }}
                </td>

                <!-- STOCK -->
                <td>
                  <v-chip
                    :color="getStockColor(product.stock)"
                    variant="tonal"
                    size="small"
                  >
                    {{ product.stock }}
                  </v-chip>
                </td>

                <!-- STATUS -->
                <td>
                  <v-chip
                    :color="product.is_active ? 'success' : 'error'"
                    variant="tonal"
                    size="small"
                  >
                    {{ product.is_active ? "Active" : "Inactive" }}
                  </v-chip>
                </td>

                <!-- ACTIONS -->
                <td>
                  <div class="d-flex justify-center ga-1">
                    <!-- VIEW -->
                    <v-btn
                      icon="mdi-eye"
                      variant="text"
                      color="info"
                      size="small"
                      @click="viewProduct(product)"
                    />

                    <!-- EDIT -->
                    <v-btn
                      icon="mdi-pencil"
                      variant="text"
                      color="primary"
                      size="small"
                      @click="openEditDialog(product)"
                    />

                    <!-- DELETE -->
                    <v-btn
                      icon="mdi-delete"
                      variant="text"
                      color="error"
                      size="small"
                      @click="openDeleteDialog(product)"
                    />
                  </div>
                </td>
              </tr>
            </tbody>
          </v-table>
        </v-card>

        <!-- ========================================
             CREATE / EDIT DIALOG
        ========================================= -->
        <v-dialog v-model="productDialog" max-width="700" persistent>
          <v-card rounded="xl">
            <!-- TITLE -->
            <v-card-title
              class="d-flex align-center justify-space-between pa-5"
            >
              <span class="text-h5 font-weight-bold">
                {{ isEditing ? "Edit Product" : "Add Product" }}
              </span>

              <v-btn
                icon="mdi-close"
                variant="text"
                @click="closeProductDialog"
              />
            </v-card-title>

            <v-divider />

            <!-- FORM -->
            <v-card-text class="pa-6">
              <v-form ref="productForm" @submit.prevent="saveProduct">
                <!-- ==================================
                     IMAGE
                =================================== -->
                <div class="mb-6">
                  <div class="text-subtitle-2 font-weight-bold mb-2">
                    Product Image
                  </div>

                  <!-- PREVIEW -->
                  <div
                    v-if="imagePreview || currentImage"
                    class="image-preview mb-4"
                  >
                    <v-img
                      :src="imagePreview || getImageUrl(currentImage)"
                      height="220"
                      contain
                    />
                  </div>

                  <!-- FILE INPUT -->
                  <v-file-input
                    v-model="selectedFile"
                    label="Select Product Image"
                    variant="outlined"
                    prepend-icon=""
                    prepend-inner-icon="mdi-camera"
                    accept="image/*"
                    show-size
                    clearable
                    @update:model-value="handleImageChange"
                  />
                </div>

                <!-- ==================================
                     PRODUCT NAME
                =================================== -->
                <v-text-field
                  v-model="form.name"
                  label="Product Name"
                  variant="outlined"
                  prepend-inner-icon="mdi-package-variant"
                  :rules="[(value) => !!value || 'Product name is required']"
                  required
                />

                <!-- ==================================
                     DESCRIPTION
                =================================== -->
                <v-textarea
                  v-model="form.description"
                  label="Description"
                  variant="outlined"
                  prepend-inner-icon="mdi-text"
                  rows="3"
                />

                <!-- ==================================
                     PRICE + STOCK
                =================================== -->
                <v-row>
                  <!-- PRICE -->
                  <v-col cols="12" md="6">
                    <v-text-field
                      v-model.number="form.price"
                      label="Price"
                      type="number"
                      min="0"
                      step="0.01"
                      variant="outlined"
                      prepend-inner-icon="mdi-currency-php"
                      :rules="priceRules"
                    />
                  </v-col>

                  <!-- STOCK -->
                  <v-col cols="12" md="6">
                    <v-text-field
                      v-model.number="form.stock"
                      label="Stock"
                      type="number"
                      min="0"
                      variant="outlined"
                      prepend-inner-icon="mdi-warehouse"
                      :rules="stockRules"
                    />
                  </v-col>
                </v-row>

                <!-- ==================================
                     CATEGORY
                =================================== -->
                <v-select
                  v-model="form.category"
                  :items="categories"
                  label="Category"
                  variant="outlined"
                  prepend-inner-icon="mdi-shape"
                  clearable
                />

                <!-- ==================================
                     ACTIVE
                =================================== -->
                <v-switch
                  v-model="form.is_active"
                  label="Active Product"
                  color="success"
                  inset
                />
              </v-form>
            </v-card-text>

            <v-divider />

            <!-- ACTION BUTTONS -->
            <v-card-actions class="pa-5">
              <v-spacer />

              <v-btn variant="text" @click="closeProductDialog"> Cancel </v-btn>

              <v-btn
                color="primary"
                :loading="saving"
                :disabled="saving"
                @click="saveProduct"
              >
                {{ isEditing ? "Update Product" : "Save Product" }}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <!-- ========================================
             VIEW PRODUCT DIALOG
        ========================================= -->
        <v-dialog v-model="viewDialog" max-width="600">
          <v-card v-if="selectedProduct" rounded="xl">
            <!-- IMAGE -->
            <v-img
              v-if="selectedProduct.image"
              :src="getImageUrl(selectedProduct.image)"
              height="320"
              contain
              class="product-view-image"
            />

            <div
              v-else
              class="d-flex align-center justify-center bg-grey-lighten-3"
              style="height: 250px"
            >
              <v-icon icon="mdi-image-off" size="80" color="grey" />
            </div>

            <!-- INFORMATION -->
            <v-card-text class="pa-6">
              <div class="d-flex align-start justify-space-between ga-4">
                <div>
                  <h2 class="text-h5 font-weight-bold">
                    {{ selectedProduct.name }}
                  </h2>

                  <v-chip
                    class="mt-2"
                    color="primary"
                    variant="tonal"
                    size="small"
                  >
                    {{ selectedProduct.category || "Uncategorized" }}
                  </v-chip>
                </div>

                <!-- STATUS -->
                <v-chip
                  :color="selectedProduct.is_active ? 'success' : 'error'"
                  variant="tonal"
                >
                  {{ selectedProduct.is_active ? "Active" : "Inactive" }}
                </v-chip>
              </div>

              <!-- DESCRIPTION -->
              <p class="mt-5 text-medium-emphasis">
                {{ selectedProduct.description || "No product description." }}
              </p>

              <v-divider class="my-5" />

              <!-- PRICE + STOCK -->
              <v-row>
                <v-col cols="6">
                  <div class="text-caption text-medium-emphasis">Price</div>

                  <div class="text-h5 font-weight-bold text-primary">
                    {{ formatPrice(selectedProduct.price) }}
                  </div>
                </v-col>

                <v-col cols="6">
                  <div class="text-caption text-medium-emphasis">Stock</div>

                  <div class="text-h6 font-weight-bold">
                    {{ selectedProduct.stock }}
                    units
                  </div>
                </v-col>
              </v-row>
            </v-card-text>

            <!-- CLOSE -->
            <v-card-actions class="pa-5 pt-0">
              <v-spacer />

              <v-btn
                color="primary"
                variant="tonal"
                @click="viewDialog = false"
              >
                Close
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <!-- ========================================
             DELETE DIALOG
        ========================================= -->
        <v-dialog v-model="deleteDialog" max-width="450">
          <v-card rounded="xl">
            <v-card-text class="text-center pa-8">
              <v-avatar color="error" variant="tonal" size="70" class="mb-4">
                <v-icon icon="mdi-delete-alert" size="38" />
              </v-avatar>

              <h2 class="text-h5 font-weight-bold">Delete Product?</h2>

              <p class="text-medium-emphasis mt-3">
                Are you sure you want to delete

                <strong>
                  {{ selectedProduct?.name }} </strong
                >?
              </p>

              <p class="text-error mt-2">This action cannot be undone.</p>
            </v-card-text>

            <v-card-actions class="pa-5 pt-0">
              <v-btn variant="text" block @click="deleteDialog = false">
                Cancel
              </v-btn>

              <v-btn
                color="error"
                block
                :loading="deleting"
                @click="deleteProduct"
              >
                Delete
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>

        <!-- ========================================
             SNACKBAR
        ========================================= -->
        <v-snackbar
          v-model="snackbar.show"
          :color="snackbar.color"
          location="bottom right"
          timeout="3000"
        >
          {{ snackbar.message }}

          <template #actions>
            <v-btn variant="text" @click="snackbar.show = false"> Close </v-btn>
          </template>
        </v-snackbar>
      </v-container>
    </v-main>
  </v-layout>
</template>

<script setup lang="ts">
// @ts-nocheck

// =============================================
// PAGE SETTINGS
// =============================================

definePageMeta({
  layout: false,
});

useHead({
  title: "Product Management | ShopKart",
});

// =============================================
// CONFIGURATION
// =============================================

const config = useRuntimeConfig();

const STRAPI_URL = config.public.strapiUrl;

// =============================================
// PRODUCT DATA
// =============================================

const products = ref([]);

const loading = ref(false);

const search = ref("");

// =============================================
// DIALOGS
// =============================================

const productDialog = ref(false);

const viewDialog = ref(false);

const deleteDialog = ref(false);

// =============================================
// SELECTED PRODUCT
// =============================================

const selectedProduct = ref(null);

const isEditing = ref(false);

// =============================================
// LOADING STATES
// =============================================

const saving = ref(false);

const deleting = ref(false);

// =============================================
// PRODUCT FORM
// =============================================

const productForm = ref(null);

const form = reactive({
  name: "",

  description: "",

  price: 0,

  stock: 0,

  category: "",

  is_active: true,
});

// =============================================
// IMAGE
// =============================================

const selectedFile = ref(null);

const imagePreview = ref(null);

const currentImage = ref(null);

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

// =============================================
// VALIDATION
// =============================================

const priceRules = [(value) => value >= 0 || "Price cannot be negative"];

const stockRules = [(value) => value >= 0 || "Stock cannot be negative"];

// =============================================
// SNACKBAR
// =============================================

const snackbar = reactive({
  show: false,

  message: "",

  color: "success",
});

// =============================================
// FILTERED PRODUCTS
// =============================================

const filteredProducts = computed(() => {
  /*
      Return all products when
      the search field is empty.
    */

  if (!search.value) {
    return products.value;
  }

  /*
      Convert search text
      to lowercase.
    */

  const keyword = search.value.toLowerCase().trim();

  /*
      Search product name,
      description and category.
    */

  return products.value.filter((product) => {
    const name = product.name?.toLowerCase() || "";

    const description = product.description?.toLowerCase() || "";

    const category = product.category?.toLowerCase() || "";

    return (
      name.includes(keyword) ||
      description.includes(keyword) ||
      category.includes(keyword)
    );
  });
});

// =============================================
// READ PRODUCTS
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
    console.error("Error fetching products:", error);

    showSnackbar("Unable to load products.", "error");
  } finally {
    loading.value = false;
  }
}

// =============================================
// OPEN CREATE DIALOG
// =============================================

function openCreateDialog() {
  resetForm();

  isEditing.value = false;

  selectedProduct.value = null;

  productDialog.value = true;
}

// =============================================
// OPEN EDIT DIALOG
// =============================================

function openEditDialog(product) {
  selectedProduct.value = product;

  isEditing.value = true;

  form.name = product.name || "";

  form.description = product.description || "";

  form.price = Number(product.price) || 0;

  form.stock = Number(product.stock) || 0;

  form.category = product.category || "";

  form.is_active = product.is_active ?? true;

  currentImage.value = product.image || null;

  selectedFile.value = null;

  imagePreview.value = null;

  productDialog.value = true;
}

// =============================================
// VIEW PRODUCT
// =============================================

function viewProduct(product) {
  selectedProduct.value = product;

  viewDialog.value = true;
}

// =============================================
// SAVE PRODUCT
// CREATE + UPDATE
// =============================================

async function saveProduct() {
  /*
    Validate Vuetify form.
  */

  const validation = await productForm.value?.validate();

  if (validation && !validation.valid) {
    return;
  }

  saving.value = true;

  try {
    /*
      Decide whether this is
      CREATE or UPDATE.
    */

    if (isEditing.value) {
      await updateProduct();
    } else {
      await createProduct();
    }

    /*
      Close dialog after
      successful save.
    */

    closeProductDialog();

    /*
      Reload products from
      Strapi.
    */

    await getProducts();
  } catch (error) {
    console.error("Error saving product:", error);

    showSnackbar(getErrorMessage(error), "error");
  } finally {
    saving.value = false;
  }
}

// =============================================
// CREATE PRODUCT
// =============================================

async function createProduct() {
  /*
    STEP 1:
    Create the product.
  */

  const response = await $fetch(`${STRAPI_URL}/api/products`, {
    method: "POST",

    body: {
      data: {
        name: form.name,

        description: form.description,

        price: Number(form.price),

        stock: Number(form.stock),

        category: form.category,

        is_active: form.is_active,
      },
    },
  });

  /*
    STEP 2:
    Upload the image and
    attach it to the newly
    created product.
  */

  if (selectedFile.value && response.data) {
    await uploadProductImage(response.data.id);
  }

  showSnackbar("Product created successfully.", "success");
}

// =============================================
// UPDATE PRODUCT
// =============================================

async function updateProduct() {
  if (!selectedProduct.value) {
    return;
  }

  /*
    Update using Strapi 5
    documentId.
  */

  const response = await $fetch(
    `${STRAPI_URL}/api/products/${selectedProduct.value.documentId}`,

    {
      method: "PUT",

      body: {
        data: {
          name: form.name,

          description: form.description,

          price: Number(form.price),

          stock: Number(form.stock),

          category: form.category,

          is_active: form.is_active,
        },
      },
    },
  );

  /*
    Upload a replacement
    image only when another
    image was selected.
  */

  if (selectedFile.value && response.data) {
    await uploadProductImage(response.data.id);
  }

  showSnackbar("Product updated successfully.", "success");
}

// =============================================
// UPLOAD PRODUCT IMAGE
// =============================================

async function uploadProductImage(productId) {
  const file = getSelectedFile();

  if (!file) {
    return;
  }

  const formData = new FormData();

  /*
    Actual image file.
  */

  formData.append("files", file);

  /*
    Product Content Type UID.
  */

  formData.append("ref", "api::product.product");

  /*
    Strapi Product ID.
  */

  formData.append("refId", String(productId));

  /*
    Media field name.
  */

  formData.append("field", "image");

  await $fetch(`${STRAPI_URL}/api/upload`, {
    method: "POST",

    body: formData,
  });
}

// =============================================
// OPEN DELETE DIALOG
// =============================================

function openDeleteDialog(product) {
  selectedProduct.value = product;

  deleteDialog.value = true;
}

// =============================================
// DELETE PRODUCT
// =============================================

async function deleteProduct() {
  if (!selectedProduct.value) {
    return;
  }

  deleting.value = true;

  try {
    /*
      Delete using
      Strapi documentId.
    */

    await $fetch(
      `${STRAPI_URL}/api/products/${selectedProduct.value.documentId}`,

      {
        method: "DELETE",
      },
    );

    deleteDialog.value = false;

    showSnackbar("Product deleted successfully.", "success");

    /*
      Reload products.
    */

    await getProducts();
  } catch (error) {
    console.error("Delete error:", error);

    showSnackbar(getErrorMessage(error), "error");
  } finally {
    deleting.value = false;
  }
}

// =============================================
// IMAGE INPUT
// =============================================

function handleImageChange(value) {
  const file = getFileFromValue(value);

  /*
    No image selected.
  */

  if (!file) {
    if (imagePreview.value) {
      URL.revokeObjectURL(imagePreview.value);
    }

    imagePreview.value = null;

    return;
  }

  /*
    Remove previous temporary
    image preview.
  */

  if (imagePreview.value) {
    URL.revokeObjectURL(imagePreview.value);
  }

  /*
    Create new preview URL.
  */

  imagePreview.value = URL.createObjectURL(file);
}

// =============================================
// GET SELECTED FILE
// =============================================

function getSelectedFile() {
  return getFileFromValue(selectedFile.value);
}

// =============================================
// NORMALIZE FILE INPUT
// =============================================

function getFileFromValue(value) {
  if (!value) {
    return null;
  }

  /*
    Single File
  */

  if (value instanceof File) {
    return value;
  }

  /*
    File Array
  */

  if (Array.isArray(value) && value.length > 0) {
    return value[0];
  }

  return null;
}

// =============================================
// GET PRODUCT IMAGE URL
// =============================================

function getImageUrl(image) {
  if (!image?.url) {
    return "";
  }

  /*
    Cloudinary or any
    remote provider.
  */

  if (image.url.startsWith("http://") || image.url.startsWith("https://")) {
    return image.url;
  }

  /*
    Local Strapi Media URL.
  */

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
// STOCK COLOR
// =============================================

function getStockColor(stock) {
  /*
    No stock.
  */

  if (Number(stock) === 0) {
    return "error";
  }

  /*
    Low stock.
  */

  if (Number(stock) <= 5) {
    return "warning";
  }

  /*
    Available stock.
  */

  return "success";
}

// =============================================
// RESET FORM
// =============================================

function resetForm() {
  form.name = "";

  form.description = "";

  form.price = 0;

  form.stock = 0;

  form.category = "";

  form.is_active = true;

  selectedFile.value = null;

  currentImage.value = null;

  /*
    Remove temporary preview.
  */

  if (imagePreview.value) {
    URL.revokeObjectURL(imagePreview.value);
  }

  imagePreview.value = null;
}

// =============================================
// CLOSE PRODUCT DIALOG
// =============================================

function closeProductDialog() {
  productDialog.value = false;

  resetForm();
}

// =============================================
// ERROR MESSAGE
// =============================================

function getErrorMessage(error) {
  return (
    error?.data?.error?.message ||
    error?.data?.message ||
    error?.message ||
    "Something went wrong."
  );
}

// =============================================
// SHOW SNACKBAR
// =============================================

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
   PAGE LAYOUT
============================================= */

.product-layout {
  min-height: 100vh;
}

.product-page {
  min-height: 100vh;

  background: #f5f7fb;
}

/* =============================================
   APP BAR
============================================= */

.shop-header {
  background: #ffffff !important;

  border-bottom: 1px solid rgba(0, 0, 0, 0.08);
}

.logo-link {
  text-decoration: none;

  white-space: nowrap;

  cursor: pointer;
}

/* =============================================
   PRODUCT TABLE
============================================= */

.product-table-card {
  overflow-x: auto;
}

/* =============================================
   PRODUCT DESCRIPTION
============================================= */

.description-text {
  max-width: 320px;

  overflow: hidden;

  text-overflow: ellipsis;

  white-space: nowrap;

  font-size: 13px;
}

/* =============================================
   IMAGE PREVIEW
============================================= */

.image-preview {
  overflow: hidden;

  border: 1px solid rgba(0, 0, 0, 0.12);

  border-radius: 12px;

  background: #f5f5f5;
}

/* =============================================
   PRODUCT VIEW IMAGE
============================================= */

.product-view-image {
  background: #f8f9fa;
}

/* =============================================
   MOBILE
============================================= */

@media (max-width: 700px) {
  .shop-header :deep(.v-container) {
    padding-left: 16px;

    padding-right: 16px;
  }

  .shop-header .v-btn {
    min-width: auto;
  }

  .product-page {
    overflow-x: hidden;
  }

  .description-text {
    max-width: 180px;
  }
}
</style>
