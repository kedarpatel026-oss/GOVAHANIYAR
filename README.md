name: govahaniyar
description: Wholesale Marketplace App
publish_to: "none"

environment:
  sdk: ">=3.0.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.25.4
  firebase_auth: ^4.17.8
  cloud_firestore: ^4.15.8

flutter:
  uses-material-design: true
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/login.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const GovahaniyarApp());
}

class GovahaniyarApp extends StatelessWidget {
  const GovahaniyarApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'GOVAHANIYAR',
      theme: ThemeData(primarySwatch: Colors.green),
      home: const LoginScreen(),
      debugShowCheckedModeBanner: false,
    );
  }
}
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'buyer_home.dart';
import 'wholesaler_home.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  String role = "buyer";
  final email = TextEditingController();
  final password = TextEditingController();

  login() async {
    await FirebaseAuth.instance.signInWithEmailAndPassword(
      email: email.text,
      password: password.text,
    );

    if (role == "buyer") {
      Navigator.pushReplacement(
          context, MaterialPageRoute(builder: (_) => const BuyerHome()));
    } else {
      Navigator.pushReplacement(
          context, MaterialPageRoute(builder: (_) => const WholesalerHome()));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("GOVAHANIYAR Login")),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(controller: email, decoration: const InputDecoration(labelText: "Email")),
            TextField(controller: password, decoration: const InputDecoration(labelText: "Password"), obscureText: true),
            DropdownButton<String>(
              value: role,
              items: const [
                DropdownMenuItem(value: "buyer", child: Text("Buyer")),
                DropdownMenuItem(value: "wholesaler", child: Text("Wholesaler")),
              ],
              onChanged: (v) => setState(() => role = v!),
            ),
            ElevatedButton(onPressed: login, child: const Text("Login")),
          ],
        ),
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class BuyerHome extends StatelessWidget {
  const BuyerHome({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("GOVAHANIYAR - Products")),
      body: StreamBuilder(
        stream: FirebaseFirestore.instance.collection("products").snapshots(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return const Center(child: CircularProgressIndicator());

          return ListView(
            children: snapshot.data!.docs.map((doc) {
              return Card(
                child: ListTile(
                  title: Text(doc['name']),
                  subtitle: Text("₹${doc['price']}"),
                  trailing: ElevatedButton(
                    child: const Text("Order"),
                    onPressed: () {
                      FirebaseFirestore.instance.collection("orders").add({
                        "product": doc['name'],
                        "price": doc['price'],
                        "commission": doc['price'] * 0.10,
                        "status": "requested"
                      });
                    },
                  ),
                ),
              );
            }).toList(),
          );
        },
      ),
    );
  }
}
import 'package:flutter/material.dart';
import 'add_product.dart';

class WholesalerHome extends StatelessWidget {
  const WholesalerHome({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Wholesaler Dashboard")),
      floatingActionButton: FloatingActionButton(
        child: const Icon(Icons.add),
        onPressed: () {
          Navigator.push(
              context, MaterialPageRoute(builder: (_) => const AddProduct()));
        },
      ),
      body: const Center(child: Text("Add products & receive orders")),
    );
  }
}
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class AddProduct extends StatefulWidget {
  const AddProduct({super.key});

  @override
  State<AddProduct> createState() => _AddProductState();
}

class _AddProductState extends State<AddProduct> {
  final name = TextEditingController();
  final price = TextEditingController();

  addProduct() async {
    await FirebaseFirestore.instance.collection("products").add({
      "name": name.text,
      "price": double.parse(price.text),
    });
    Navigator.pop(context);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("Add Product")),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(controller: name, decoration: const InputDecoration(labelText: "Product Name")),
            TextField(controller: price, decoration: const InputDecoration(labelText: "Price")),
            ElevatedButton(onPressed: addProduct, child: const Text("Save")),
          ],
        ),
      ),
    );
  }
}
products
 └── name
 └── price

orders
 └── product
 └── price
 └── commission (10%)
 └── status
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.25.4
  firebase_auth: ^4.17.8
  cloud_firestore: ^4.15.8
import 'package:flutter/material.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'buyer_home.dart';
import 'wholesaler_home.dart';

class LoginScreen extends StatefulWidget {
  const LoginScreen({super.key});

  @override
  State<LoginScreen> createState() => _LoginScreenState();
}

class _LoginScreenState extends State<LoginScreen> {
  final phoneController = TextEditingController();
  final otpController = TextEditingController();

  String verificationId = "";
  String role = "buyer";

  sendOTP() async {
    await FirebaseAuth.instance.verifyPhoneNumber(
      phoneNumber: "+91${phoneController.text}",
      verificationCompleted: (credential) {},
      verificationFailed: (e) {
        ScaffoldMessenger.of(context)
            .showSnackBar(SnackBar(content: Text(e.message!)));
      },
      codeSent: (verId, _) {
        setState(() => verificationId = verId);
      },
      codeAutoRetrievalTimeout: (verId) {
        verificationId = verId;
      },
    );
  }

  verifyOTP() async {
    PhoneAuthCredential credential = PhoneAuthProvider.credential(
      verificationId: verificationId,
      smsCode: otpController.text,
    );

    await FirebaseAuth.instance.signInWithCredential(credential);

    if (role == "buyer") {
      Navigator.pushReplacement(
          context, MaterialPageRoute(builder: (_) => const BuyerHome()));
    } else {
      Navigator.pushReplacement(
          context, MaterialPageRoute(builder: (_) => const WholesalerHome()));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text("GOVAHANIYAR Login")),
      body: Padding(
        padding: const EdgeInsets.all(16),
        child: Column(
          children: [
            TextField(
              controller: phoneController,
              keyboardType: TextInputType.phone,
              decoration: const InputDecoration(labelText: "Phone Number"),
            ),
            ElevatedButton(onPressed: sendOTP, child: const Text("Send OTP")),

            TextField(
              controller: otpController,
              keyboardType: TextInputType.number,
              decoration: const InputDecoration(labelText: "Enter OTP"),
            ),

            DropdownButton<String>(
              value: role,
              items: const [
                DropdownMenuItem(value: "buyer", child: Text("Buyer")),
                DropdownMenuItem(value: "wholesaler", child: Text("Wholesaler")),
              ],
              onChanged: (v) => setState(() => role = v!),
            ),

            ElevatedButton(onPressed: verifyOTP, child: const Text("Verify & Login")),
          ],
        ),
      ),
    );
  }
}
FirebaseFirestore.instance.collection("users").doc(
  FirebaseAuth.instance.currentUser!.uid
).set({
  "phone": phoneController.text,
  "role": role,
});
users
 └── uid
     ├── phone
     ├── role (buyer / wholesaler)

products
 ├── name
 ├── price
 ├── wholesalerId

orders
 ├── product
 ├── buyerId
 ├── wholesalerId
 ├── totalPrice
 ├── commission (10%)
 ├── status
