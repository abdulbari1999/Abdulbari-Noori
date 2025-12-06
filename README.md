Stock Manager - Final Version (Ready for Deploymen...

<!DOCTYPE html>
<html lang="en">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>Modern Stock Inventory</title>
   <!-- Load Tailwind CSS for styling -->
   <script src="https://cdn.tailwindcss.com"></script>

   <style>
       /* Custom styles for a modern, mobile-friendly look */
       body {
           font-family: 'Inter', sans-serif;
           background-color: #eef2ff; /* Light Lavender background */
       }
       .container-wrapper {
           max-width: 600px;
           margin: 0 auto;
           min-height: 100vh;
       }
       .card {
           background: white;
           /* Deeper shadow for a modern, lifted look */
           box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
           border-radius: 16px; /* More rounded corners */
       }
       .btn-base {
           transition: all 0.2s ease-in-out;
           border-radius: 12px;
           font-weight: 600;
       }
       .btn-base:hover {
           transform: translateY(-2px);
       }
       /* Style for the quick adjust buttons (mobile touch targets) */
       .quantity-control button {
           width: 48px; /* Slightly larger touch target */
           height: 48px;
           font-size: 1.4rem;
           font-weight: bold;
           display: flex;
           align-items: center;
           justify-content: center;
           border-radius: 8px;
           transition: background-color 0.15s, transform 0.1s;
           flex-shrink: 0;
           cursor: pointer;
           box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
       }
       .quantity-control button:active {
           transform: scale(0.95);
       }
   </style>
</head>
<body>
   <div class="container-wrapper p-4 pt-8 pb-10">
       <h1 class="text-3xl font-extrabold text-indigo-800 text-center mb-8">
           <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 inline-block mr-2" viewBox="0 0 20 20" fill="currentColor">
             <path d="M7 3a1 1 0 000 2h6a1 1 0 100-2H7zM4 6a1 1 0 011-1h10a1 1 0 011 1v10a1 1 0 01-1 1H5a1 1 0 01-1-1V6z" clip-rule="evenodd" fill-rule="evenodd" />
           </svg>
           Stock Manager
       </h1>

       <!-- User ID Display (Crucial for Firestore security rules) -->
       <div class="mb-6 text-center text-xs text-gray-500 p-3 bg-white rounded-xl shadow-inner border-t-2 border-indigo-200">
           Current User ID: <span id="userIdDisplay" class="font-mono text-indigo-600 font-semibold">Loading...</span>
       </div>

       <!-- Add/Update Product Form with quick adjust buttons -->
       <div class="card p-6 mb-8">
           <h2 class="text-xl font-bold mb-5 text-indigo-700" id="formTitle">Add New Product</h2>
           <form id="productForm" class="space-y-4">
               <input type="hidden" id="productId">
               <input type="hidden" id="originalQuantity">
               <div>
                   <label for="productName" class="block text-sm font-medium text-gray-600 mb-1">Product Name</label>
                   <input type="text" id="productName" required class="w-full p-3 border border-gray-300 rounded-xl focus:ring-indigo-500 focus:border-indigo-500 shadow-sm" placeholder="e.g., Premium Widget V2">
               </div>
               
               <!-- Quantity Adjustment Input Group -->
               <div>
                   <label for="productQuantity" class="block text-sm font-medium text-gray-600 mb-1">
                       Quantity (pcs) - Supports simple math (e.g., 5*3)
                   </label>
                   <div class="flex space-x-3 quantity-control items-stretch">
                       <!-- Decrement Button -->
                       <button type="button" id="decrementBtn" class="bg-red-500 text-white hover:bg-red-600 shadow-md btn-base" aria-label="Decrease Quantity">
                           -
                       </button>
                       <!-- Quantity Input Field -->
                       <input type="text" id="productQuantity" required value="0" class="flex-grow p-3 border border-gray-300 rounded-xl text-center focus:ring-indigo-500 focus:border-indigo-500 shadow-sm text-lg font-mono">
                       <!-- Increment Button -->
                       <button type="button" id="incrementBtn" class="bg-green-500 text-white hover:bg-green-600 shadow-md btn-base" aria-label="Increase Quantity">
                           +
                       </button>
                   </div>
               </div>

               <!-- Notes Text Area -->
               <div>
                   <label for="productNotes" class="block text-sm font-medium text-gray-600 mb-1">Notes (Optional)</label>
                   <textarea id="productNotes" rows="3" class="w-full p-3 border border-gray-300 rounded-xl focus:ring-indigo-500 focus:border-indigo-500 shadow-sm" placeholder="e.g., Warehouse A, requires cold storage, Supplier X batch number 3..."></textarea>
               </div>
               <!-- END Notes Text Area -->

               <button type="submit" id="submitButton" class="btn-base w-full py-3 bg-indigo-600 text-white text-lg hover:bg-indigo-700 shadow-lg shadow-indigo-300/50">
                   Add Product to Stock
               </button>
               <button type="button" id="cancelEditBtn" class="w-full py-3 text-sm text-gray-600 hover:text-gray-900 hidden transition duration-150">
                   Cancel Edit
               </button>
           </form>
       </div>

       <!-- Stock List and Export Button -->
       <div class="card p-6">
           <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-5 border-b pb-4 border-gray-100">
               <h2 class="text-xl font-bold text-indigo-700 mb-2 sm:mb-0">
                   Current Stock (<span id="totalItems" class="text-green-600">0</span> items)
               </h2>
               <button id="exportCsvBtn" class="btn-base flex items-center space-x-2 py-2 px-4 bg-green-500 text-white text-sm hover:bg-green-600 shadow-md shadow-green-300/50">
                   <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                       <path fill-rule="evenodd" d="M3 17V3h8v4.5a.5.5 0 01-.5.5h-5a.5.5 0 01-.5-.5V4h-2v12h10v-3.5a.5.5 0 011 0v4a.5.5 0 01-.5.5H3.5a.5.5 0 01-.5-.5z" clip-rule="evenodd" />
                   </svg>
                   <span>Export CSV</span>
               </button>
           </div>
           
           <div id="loadingIndicator" class="text-center text-gray-500 py-6 hidden">Loading stock data...</div>
           <div id="stockListContainer">
               <ul id="stockList" class="min-h-[100px] divide-y divide-gray-100">
                   <li class="p-4 text-gray-500 text-center">No products in stock yet.</li>
               </ul>
           </div>
       </div>
   </div>

   <!-- Confirmation Modal Structure (for Delete/Error) -->
   <div id="modalOverlay" class="fixed inset-0 bg-black bg-opacity-60 hidden items-center justify-center p-4 z-50 transition-opacity duration-300">
       <div id="modalContent" class="bg-white p-6 rounded-2xl shadow-2xl w-full max-w-sm space-y-4 transform scale-100 transition-transform duration-300">
           <h3 class="text-xl font-bold text-gray-800" id="modalTitle"></h3>
           <p class="text-gray-600" id="modalMessage"></p>
           <div class="flex justify-end space-x-3 pt-2">
               <button id="modalCancelBtn" class="px-4 py-2 text-sm font-medium text-gray-700 bg-gray-200 rounded-xl hover:bg-gray-300 transition duration-150">Close</button>
               <button id="modalConfirmBtn" class="px-4 py-2 text-sm font-medium text-white bg-red-600 rounded-xl hover:bg-red-700 transition duration-150">Confirm Delete</button>
           </div>
       </div>
   </div>


   <script type="module">
       // Import Firebase components
       import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
       import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
       import { getFirestore, doc, addDoc, setDoc, deleteDoc, collection, query, onSnapshot, serverTimestamp, setLogLevel, where, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

       // Global variables provided by the environment
       const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
       const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
       const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

       // Firebase instances
       let app, db, auth;
       let userId = null;
       let isAuthReady = false;
       
       // Global variable to hold the current product list (set by onSnapshot)
       let currentProducts = [];

       // --- DOM Elements ---
       const stockListEl = document.getElementById('stockList');
       const totalItemsEl = document.getElementById('totalItems');
       const form = document.getElementById('productForm');
       const formTitle = document.getElementById('formTitle');
       const productNameInput = document.getElementById('productName');
       const productQuantityInput = document.getElementById('productQuantity');
       const productNotesInput = document.getElementById('productNotes');
       const productIdInput = document.getElementById('productId');
       const submitButton = document.getElementById('submitButton');
       const exportCsvBtn = document.getElementById('exportCsvBtn');
       const loadingIndicator = document.getElementById('loadingIndicator');
       const userIdDisplay = document.getElementById('userIdDisplay');
       const incrementBtn = document.getElementById('incrementBtn');
       const decrementBtn = document.getElementById('decrementBtn');
       const cancelEditBtn = document.getElementById('cancelEditBtn');
       
       // Modal elements
       const modalOverlay = document.getElementById('modalOverlay');
       const modalTitle = document.getElementById('modalTitle');
       const modalMessage = document.getElementById('modalMessage');
       const modalCancelBtn = document.getElementById('modalCancelBtn');
       const modalConfirmBtn = document.getElementById('modalConfirmBtn');

       
       // --- QUANTITY CALCULATION LOGIC ---

       /**
        * Safely evaluates a simple arithmetic expression.
        * @param {string} expression - The arithmetic expression string (e.g., "10*5+2").
        * @returns {number|NaN} The result of the expression or NaN if invalid/negative.
        */
       const safeEvalExpression = (expression) => {
           // 1. Clean expression: remove spaces
           const cleaned = expression.replace(/\s/g, '');

           // 2. Simple validation: ensures the expression only contains numbers,
           //    basic operators (+, -, *, /), and parentheses.
           if (!/^[\-\(]?[0-9\.\+\-\*\/\(\)]+$/.test(cleaned)) {
               return NaN; // Not a valid arithmetic expression
           }

           try {
               // Use the Function constructor as a limited, safer alternative to eval for arithmetic
               const result = new Function('return ' + cleaned)();

               // Check if the result is a finite number and non-negative (for quantity)
               if (Number.isFinite(result) && result >= 0) {
                   // Return result, rounded to 2 decimal places if it's a float
                   return parseFloat(result.toFixed(2));
               }
               return NaN;
           } catch (e) {
               // Calculation failed (e.g., division by zero, incomplete expression)
               return NaN;
           }
       };


       // Event listener for quantity input to auto-calculate
       productQuantityInput.addEventListener('input', (e) => {
           const expression = e.target.value;
           const result = safeEvalExpression(expression);

           // If calculation is successful and the result is different from the expression
           // (prevents infinite loop if the user types a number '5' which evaluates to '5')
           if (!isNaN(result) && String(result) !== expression) {
               // Update the input field with the calculated result
               e.target.value = result;
           }
           // If NaN, leave the expression as is so the user can continue typing.
       });


       // --- QUANTITY BUTTON LISTENERS ---
       
       incrementBtn.addEventListener('click', () => {
           let currentQty = parseInt(productQuantityInput.value || '0', 10);
           if (isNaN(currentQty)) currentQty = 0;
           productQuantityInput.value = currentQty + 1;
       });

       decrementBtn.addEventListener('click', () => {
           let currentQty = parseInt(productQuantityInput.value || '0', 10);
           if (isNaN(currentQty)) currentQty = 0;
           
           if (currentQty > 0) {
               productQuantityInput.value = currentQty - 1;
           } else {
               productQuantityInput.value = 0;
           }
       });
       // --- END QUANTITY BUTTON LISTENERS ---


       // --- FIREBASE INITIALIZATION AND AUTHENTICATION ---
       try {
           setLogLevel('debug');
           app = initializeApp(firebaseConfig);
           db = getFirestore(app);
           auth = getAuth(app);
           console.log("Firebase Initialized.");

           onAuthStateChanged(auth, async (user) => {
               if (user) {
                   userId = user.uid;
                   userIdDisplay.textContent = userId;
                   isAuthReady = true;
                   console.log("User authenticated:", userId);
                   setupRealtimeListener();
               } else {
                   try {
                       if (initialAuthToken) {
                           await signInWithCustomToken(auth, initialAuthToken);
                       } else {
                           await signInAnonymously(auth);
                       }
                   } catch (error) {
                       console.error("Authentication failed:", error);
                       userIdDisplay.textContent = "Auth Error!";
                   }
               }
           });
       } catch (error) {
           console.error("Error initializing Firebase:", error);
           userIdDisplay.textContent = "Init Error!";
       }

       // --- FIRESTORE OPERATIONS ---

       const getCollectionPath = (uid) => {
           return `artifacts/${appId}/users/${uid}/stock_products`;
       };
       
       // Save or Update Product
       const saveProduct = async (name, quantity, notes, id = null) => {
           if (!isAuthReady || !userId) {
               console.error("Auth not ready. Cannot save.");
               return;
           }
           
           const quantityValue = parseInt(quantity, 10);

           if (isNaN(quantityValue) || quantityValue < 0) {
                showModal("Input Error", "The quantity is invalid or less than zero. Please check your math expression.", false);
                return;
           }

           const trimmedName = name.trim();
           // Include notes in the data being saved
           const productData = {
               name: trimmedName,
               quantity: quantityValue,
               notes: notes.trim(), // Save notes
               updatedAt: serverTimestamp(),
           };
           
           const productsCollection = collection(db, getCollectionPath(userId));

           try {
               if (!id) {
                   // --- Check for Duplicates ---
                   const existingProductQuery = query(productsCollection, where("name", "==", trimmedName));
                   const snapshot = await getDocs(existingProductQuery);
                   
                   if (!snapshot.empty) {
                       showModal("Duplicate Found", `A product named '${trimmedName}' already exists. Please edit the existing item instead of creating a duplicate.`, false);
                       resetForm();
                       return;
                   }
                   
                   // Add new product
                   productData.createdAt = serverTimestamp();
                   await addDoc(productsCollection, productData);
                   console.log("New product added.");

               } else {
                   // Update existing product
                   const productDocRef = doc(productsCollection, id);
                   await setDoc(productDocRef, productData, { merge: true });
                   console.log(`Product updated with ID: ${id}`);
               }
               
               resetForm();

           } catch (error) {
               console.error("Error saving product:", error);
               showModal("Error", `Could not save product: ${error.message}`, false);
           }
       };

       // Delete Product
       const deleteProduct = async (id) => {
            if (!isAuthReady || !userId) {
               console.error("Auth not ready. Cannot delete.");
               return;
           }
           try {
               const productDocRef = doc(db, getCollectionPath(userId), id);
               await deleteDoc(productDocRef);
               console.log(`Product deleted with ID: ${id}`);
           } catch (error) {
               console.error("Error deleting product:", error);
               showModal("Error", `Could not delete product: ${error.message}`, false);
           }
       };

       // Real-time Listener for Stock Data
       const setupRealtimeListener = () => {
           if (!isAuthReady || !userId) return;

           const productsCollectionRef = collection(db, getCollectionPath(userId));
           const q = query(productsCollectionRef);
           
           loadingIndicator.classList.remove('hidden');

           onSnapshot(q, (snapshot) => {
               loadingIndicator.classList.add('hidden');
               
               const products = [];
               snapshot.forEach(doc => {
                   const data = doc.data();
                   products.push({
                       id: doc.id,
                       name: data.name,
                       quantity: data.quantity,
                       notes: data.notes || '', // Retrieve notes
                       updatedAt: data.updatedAt ? data.updatedAt.toDate() : new Date(),
                   });
               });
               
               // Sort by name alphabetically (in-memory sort)
               products.sort((a, b) => a.name.localeCompare(b.name));
               currentProducts = products; // Store the global list for CSV export

               renderStockList(products);
               console.log("Stock list updated.");

           }, (error) => {
               console.error("Error listening to stock data:", error);
               loadingIndicator.classList.add('hidden');
               stockListEl.innerHTML = `<li class="p-4 text-red-500 text-center">Error loading data: ${error.message}</li>`;
           });
       };

       // --- UI RENDERING AND EVENTS ---

       // Render the list of products
       const renderStockList = (products) => {
           stockListEl.innerHTML = '';
           totalItemsEl.textContent = products.length;

           if (products.length === 0) {
               stockListEl.innerHTML = `<li class="p-4 text-gray-500 text-center">No products in stock yet.</li>`;
               return;
           }

           products.forEach(product => {
               const li = document.createElement('li');
               li.className = 'flex justify-between items-center p-4 hover:bg-indigo-50 transition duration-150';
               
               // Store IDs and data directly on the element for retrieval
               li.dataset.id = product.id;
               li.dataset.name = product.name;
               li.dataset.quantity = product.quantity;
               li.dataset.notes = product.notes; // Store notes

               li.innerHTML = `
                   <div class="flex-grow min-w-0 pr-4">
                       <p class="font-semibold text-gray-800 truncate">${product.name}</p>
                       <p class="text-sm text-gray-500">Stock: <span class="font-extrabold text-indigo-600">${product.quantity}</span> pcs</p>
                       ${product.notes ? `<p class="text-xs text-gray-400 italic mt-1 truncate">Notes: ${product.notes}</p>` : ''}
                   </div>
                   <div class="flex space-x-2 flex-shrink-0">
                       <button data-action="edit" class="edit-btn p-2 text-sm font-medium text-indigo-600 bg-indigo-100 rounded-full hover:bg-indigo-200 transition duration-150 active:scale-95 shadow-md" aria-label="Edit ${product.name}">
                           <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 pointer-events-none" viewBox="0 0 20 20" fill="currentColor"><path d="M17.414 2.586a2 2 0 00-2.828 0L7 10.172V13h2.828l7.586-7.586a2 2 0 000-2.828z" /><path fill-rule="evenodd" d="M2 6a2 2 0 012-2h4a1 1 0 010 2H4v10h10v-4a1 1 0 112 0v4a2 2 0 01-2 2H4a2 2 0 01-2-2V6z" clip-rule="evenodd" /></svg>
                       </button>
                       <button data-action="delete" class="delete-btn p-2 text-sm font-medium text-red-600 bg-red-100 rounded-full hover:bg-red-200 transition duration-150 active:scale-95 shadow-md" aria-label="Delete ${product.name}">
                           <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 pointer-events-none" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" /></svg>
                       </button>
                   </div>
               `;
               stockListEl.appendChild(li);
           });
       };

       // Handle form submission (Add or Update)
       form.addEventListener('submit', (e) => {
           e.preventDefault();
           const name = productNameInput.value;
           // Before saving, ensure the quantity input (which might still be an expression) is calculated
           const finalQuantity = safeEvalExpression(productQuantityInput.value);
           const notes = productNotesInput.value;
           const id = productIdInput.value || null;

           if (name.trim() && !isNaN(finalQuantity) && finalQuantity >= 0) {
               saveProduct(name, finalQuantity, notes, id); // Pass the calculated result to saveProduct
           } else {
                showModal("Input Error", "Please enter a valid product name and a quantity (number or valid math expression) of 0 or greater.", false);
           }
       });
       
       // Handle Cancel Edit Button
       cancelEditBtn.addEventListener('click', resetForm);

       // Delegate click events for Edit and Delete
       stockListEl.addEventListener('click', (e) => {
           const btn = e.target.closest('button');
           if (!btn) return;
           
           const li = btn.closest('li');
           if (!li || !li.dataset.id) return;
           
           const id = li.dataset.id;
           const name = li.dataset.name;
           const quantity = li.dataset.quantity;
           const notes = li.dataset.notes;
           const action = btn.dataset.action;

           if (action === 'edit') {
               // Populate form for editing
               productIdInput.value = id;
               productNameInput.value = name;
               // When editing, populate with the numeric quantity, not an expression
               productQuantityInput.value = parseInt(quantity, 10);
               productNotesInput.value = notes;
               formTitle.textContent = `Edit Product: ${name}`;
               submitButton.textContent = 'Save Changes';
               cancelEditBtn.classList.remove('hidden');
               productNameInput.focus();
           } else if (action === 'delete') {
               // Show confirmation modal
               showModal("Confirm Deletion", `Are you sure you want to delete '${name}' (Qty: ${quantity})? This action cannot be undone.`, true, () => {
                   deleteProduct(id);
               });
           }
       });


       // Reset the form to "Add New" state
       const resetForm = () => {
           form.reset();
           productIdInput.value = '';
           productNameInput.value = '';
           productQuantityInput.value = 0;
           productNotesInput.value = '';
           formTitle.textContent = 'Add New Product';
           submitButton.textContent = 'Add Product to Stock';
           cancelEditBtn.classList.add('hidden');
       };
       // Initialize form to reset state on load
       resetForm();


       // --- CSV EXPORT FUNCTIONALITY (Updated to include Notes) ---

       exportCsvBtn.addEventListener('click', generateCSV);

       function convertToCSV(data) {
           if (data.length === 0) return '';
           
           // Define headers - Notes added here
           const headers = ["ID", "Name", "Quantity", "Notes", "Last Updated"];
           let csv = headers.join(',') + '\n';

           data.forEach(product => {
               // Format date nicely
               const dateStr = product.updatedAt ? product.updatedAt.toLocaleDateString('en-US', {
                   year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit'
               }) : 'N/A';
               
               // Function to format/escape string for CSV
               const formatForCSV = (str) => {
                   if (!str) return '""';
                   // Escape double quotes by doubling them, and wrap the whole string in quotes
                   return `"${String(str).replace(/"/g, '""')}"`;
               };

               const row = [
                   formatForCSV(product.id),
                   formatForCSV(product.name),
                   product.quantity,
                   formatForCSV(product.notes), // Include Notes
                   formatForCSV(dateStr)
               ];
               csv += row.join(',') + '\n';
           });
           return csv;
       }

       async function generateCSV() {
           if (currentProducts.length === 0) {
               showModal("Export Error", "Cannot export. The stock list is empty.", false);
               return;
           }
           
           exportCsvBtn.textContent = 'Generating...';
           exportCsvBtn.disabled = true;

           try {
               const csvContent = convertToCSV(currentProducts);
               // Create a Blob containing the CSV data
               const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
               const url = URL.createObjectURL(blob);
               
               // Use a temporary link element to trigger the download
               const a = document.createElement('a');
               a.href = url;
               a.setAttribute('download', `Stock_Report_${new Date().toISOString().slice(0, 10)}.csv`);
               
               // Append to body, click, and remove
               document.body.appendChild(a);
               a.click();
               document.body.removeChild(a);
               URL.revokeObjectURL(url); // Clean up memory
               
               showModal("Export Success", "Stock data has been successfully prepared for download as a CSV file.", false);

           } catch (error) {
               console.error("CSV Export failed:", error);
               showModal("Export Failed", `The CSV export failed: ${error.message}`, false);
           }
           
           exportCsvBtn.textContent = 'Export CSV';
           exportCsvBtn.disabled = false;
       }


       // --- UTILITY: CUSTOM MODAL (replaces alert/confirm) ---
       let confirmActionCallback = null;

       function showModal(title, message, isConfirm, callback = null) {
           modalTitle.textContent = title;
           modalMessage.textContent = message;
           confirmActionCallback = callback;
           
           if (isConfirm) {
               modalConfirmBtn.classList.remove('hidden');
               modalCancelBtn.textContent = 'Cancel';
               modalConfirmBtn.textContent = 'Confirm Delete';
               modalConfirmBtn.classList.add('bg-red-600', 'hover:bg-red-700');
               modalConfirmBtn.classList.remove('bg-indigo-600', 'hover:bg-indigo-700');
           } else {
               // For non-confirm dialogs (like errors/success), make the confirm button an "OK" button
               modalConfirmBtn.classList.remove('hidden');
               modalConfirmBtn.classList.remove('bg-red-600', 'hover:bg-red-700');
               modalConfirmBtn.classList.add('bg-indigo-600', 'hover:bg-indigo-700');
               modalConfirmBtn.textContent = 'OK';
               modalCancelBtn.textContent = 'Close';
           }

           modalOverlay.classList.remove('hidden');
           modalOverlay.classList.add('flex');
       }

       // Modal event handlers
       modalOverlay.addEventListener('click', (e) => {
           if (e.target.id === 'modalOverlay' || e.target.id === 'modalCancelBtn') {
               modalOverlay.classList.add('hidden');
               modalOverlay.classList.remove('flex');
           }
       });

       modalConfirmBtn.addEventListener('click', () => {
           if (modalConfirmBtn.textContent === 'OK') {
               modalOverlay.classList.add('hidden');
               modalOverlay.classList.remove('flex');
           } else if (confirmActionCallback) {
               confirmActionCallback();
               modalOverlay.classList.add('hidden');
               modalOverlay.classList.remove('flex');
               confirmActionCallback = null;
           }
       });
       
       modalCancelBtn.addEventListener('click', () => {
           modalOverlay.classList.add('hidden');
           modalOverlay.classList.remove('flex');
           confirmActionCallback = null;
       });

   </script>
</body>
</html>