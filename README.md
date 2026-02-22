<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>EarnHub Premium</title>

<style>
*{margin:0;padding:0;box-sizing:border-box;font-family:'Poppins', sans-serif;}
body{background:#f0f2f5;min-height:100vh;display:flex;justify-content:center;align-items:flex-start;padding-top:30px;}
.container{width:95%;max-width:480px;background:white;border-radius:20px;box-shadow:0 20px 50px rgba(0,0,0,0.1);padding:25px;animation:fadeIn 0.5s ease;}
h2{text-align:center;margin-bottom:20px;color:#333;}
input{width:100%;padding:12px 15px;margin-top:12px;border-radius:10px;border:1px solid #ddd;outline:none;font-size:16px;}
input:focus{border-color:#ff512f;box-shadow:0 0 5px rgba(255,81,47,0.3);}
button{width:100%;padding:12px;margin-top:15px;border:none;border-radius:10px;background:linear-gradient(135deg,#ff512f,#dd2476);color:white;font-weight:bold;cursor:pointer;font-size:16px;transition:0.3s;}
button:hover{transform:scale(1.03);}
.logout{background:#333;margin-top:10px;}
.card{border-radius:15px;box-shadow:0 8px 25px rgba(0,0,0,0.08);background:white;margin-top:15px;padding:15px;transition:0.3s;}
.card:hover{transform:translateY(-3px);}
.card img{width:100%;border-radius:10px;margin-bottom:10px;}
.stat{background:linear-gradient(135deg,#ff512f,#dd2476);color:white;padding:15px;border-radius:15px;margin-top:15px;text-align:center;font-size:18px;}
#orderHistory table{width:100%;border-collapse:collapse;margin-top:10px;}
#orderHistory th, #orderHistory td{border:1px solid #ddd;padding:8px;text-align:center;}
#orderHistory th{background:#ff512f;color:white;border-radius:5px;}
.hidden{display:none;}
@keyframes fadeIn{from{opacity:0;transform:translateY(10px);}to{opacity:1;transform:translateY(0);}}
.tabs{display:flex;margin-top:15px;justify-content:space-around;}
.tabs button{flex:1;margin:0 5px;font-size:14px;}
</style>
</head>
<body>

<!-- LOGIN -->
<div class="container" id="loginBox">
    <h2>🔥 EarnHub Login</h2>
    <input type="email" id="email" placeholder="Email">
    <input type="password" id="password" placeholder="Password">
    <button onclick="login()">Login</button>
</div>

<!-- ADMIN PANEL -->
<div class="container hidden" id="adminPanel">
    <h2>📊 Admin Dashboard</h2>
    <div class="stat">
        📦 Products: <span id="totalProducts">0</span><br>
        🧾 Orders: <span id="totalOrders">0</span>
    </div>

    <div class="tabs">
        <button onclick="showTab('addProductTab')">Add Product</button>
        <button onclick="showTab('ordersTab')">Orders</button>
    </div>

    <div id="addProductTab">
        <h3>Add Product</h3>
        <input type="text" id="pname" placeholder="Product Name">
        <input type="number" id="pprice" placeholder="Price">
        <input type="text" id="pimage" placeholder="Image URL">
        <button onclick="addProduct()">Add Product</button>
    </div>

    <div id="ordersTab" class="hidden">
        <h3>Order History</h3>
        <div id="adminOrders"></div>
    </div>

    <button class="logout" onclick="logout()">Logout</button>
</div>

<!-- RESELLER PANEL -->
<div class="container hidden" id="resellerPanel">
    <h2>💰 Reseller Panel</h2>

    <div class="tabs">
        <button onclick="showTab('productsTab')">Products</button>
        <button onclick="showTab('orderHistoryTab')">Orders</button>
    </div>

    <div id="productsTab">
        <div id="productList"></div>
        <div class="stat">
            Your Earnings: ₱<span id="earnings">0</span>
        </div>
        <button onclick="withdraw()">Request Withdrawal</button>
    </div>

    <div id="orderHistoryTab" class="hidden">
        <div id="orderHistory"></div>
    </div>

    <button class="logout" onclick="logout()">Logout</button>
</div>

<!-- Firebase -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import { getAuth, signInWithEmailAndPassword, signOut, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
import { getFirestore, collection, addDoc, getDocs, doc, getDoc, updateDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "AIzaSyAzkuPuX2FmdbCEaUaPxc-2FC4GmKTKoXg",
  authDomain: "earnhub-9435f.firebaseapp.com",
  projectId: "earnhub-9435f",
  storageBucket: "earnhub-9435f.firebasestorage.app",
  messagingSenderId: "308842759840",
  appId: "1:308842759840:web:8cd3bfd0d3372c2475ebc5",
  measurementId: "G-003231CDDS"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

// Login
window.login = function(){
    console.log("Attempt login:", email.value);
    signInWithEmailAndPassword(auth, email.value, password.value)
    .then(userCredential => { console.log("Login success:", userCredential.user.uid); })
    .catch(error => { console.error("Login failed:", error.code, error.message); alert(error.message); });
}

// Logout
window.logout = function(){ signOut(auth); }

// Tabs
function showTab(id){
    document.querySelectorAll('#adminPanel > div, #resellerPanel > div').forEach(e=>e.classList.add('hidden'));
    document.getElementById(id).classList.remove('hidden');
}

// Auth state
onAuthStateChanged(auth, async (user)=>{
    if(user){
        loginBox.classList.add("hidden");
        const userDoc = await getDoc(doc(db,"users",user.uid));
        if(!userDoc.exists()){ alert("Firestore user not found! Check UID."); return; }
        const role = userDoc.data().role;
        if(role==="admin"){ 
            adminPanel.classList.remove("hidden");
            loadAdminStats(); loadAdminOrders();
        } else { 
            resellerPanel.classList.remove("hidden");
            loadProducts(user.uid); loadOrders(user.uid); loadEarnings(user.uid);
        }
    }
});

// Admin functions
async function loadAdminStats(){
    const products = await getDocs(collection(db,"products"));
    document.getElementById("totalProducts").innerText = products.size;
    const orders = await getDocs(collection(db,"orders"));
    document.getElementById("totalOrders").innerText = orders.size;
}

async function addProduct(){
    await addDoc(collection(db,"products"),{
        name:pname.value,
        price:parseInt(pprice.value),
        image:pimage.value
    });
    alert("Product Added!");
}

function loadAdminOrders(){
    onSnapshot(collection(db,"orders"), snapshot=>{
        const adminOrders = document.getElementById("adminOrders");
        adminOrders.innerHTML="";
        snapshot.forEach(docu=>{
            const data=docu.data();
            adminOrders.innerHTML+=`
            <div class="card">
                Product ID: ${data.productId}<br>
                Price: ₱${data.price}<br>
                Commission: ₱${data.commission}<br>
                Reseller ID: ${data.resellerId}
            </div>`;
        });
    });
}

// Reseller functions
function loadProducts(uid){
    onSnapshot(collection(db,"products"), snapshot=>{
        const productList = document.getElementById("productList");
        productList.innerHTML="";
        snapshot.forEach(docu=>{
            const data=docu.data();
            productList.innerHTML+=`
            <div class="card">
                <img src="${data.image}" alt="${data.name}">
                <h4>${data.name}</h4>
                <p>₱${data.price}</p>
                <button onclick="sell('${docu.id}',${data.price})">Sell</button>
            </div>`;
        });
    });
}

async function sell(productId, price){
    const user = auth.currentUser;
    const commission = price*0.20;
    await addDoc(collection(db,"orders"),{
        productId:productId, price:price, resellerId:user.uid, commission:commission, date:new Date()
    });
    const userRef = doc(db,"users",user.uid);
    const userSnap = await getDoc(userRef);
    const currentEarnings = userSnap.data().earnings||0;
    await updateDoc(userRef,{ earnings: currentEarnings+commission });
    alert("Sold! Commission added.");
}

function loadEarnings(uid){
    getDoc(doc(db,"users",uid)).then(userSnap=>{
        document.getElementById("earnings").innerText = userSnap.data().earnings||0;
    });
}

function loadOrders(uid){
    onSnapshot(collection(db,"orders"), snapshot=>{
        const orderHistory = document.getElementById("orderHistory");
        orderHistory.innerHTML="";
        snapshot.forEach(docu=>{
            const data=docu.data();
            if(data.resellerId===uid){
                orderHistory.innerHTML+=`
                <div class="card">
                    Product ID: ${data.productId}<br>
                    Price: ₱${data.price}<br>
                    Commission: ₱${data.commission}<br>
                </div>`;
            }
        });
    });
}

function withdraw(){ alert("Withdrawal request sent to Admin!"); }

</script>
</body>
</html>
