# Part-2-Assignment-Taste-Tap
Musa Mkansi
import * as React from 'react';
import { useState, createContext, useContext, useEffect } from 'react';
import { 
  ImageBackground, Text, SafeAreaView, 
  StyleSheet, TextInput, ScrollView, View, Alert, TouchableOpacity, Image, Animated
} from 'react-native';
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

// ------------------ CONTEXTS ------------------
const CartContext = createContext();
const MenuContext = createContext();

// ------------------ NAVIGATION ------------------
const Stack = createNativeStackNavigator();

// ------------------ HOME SCREEN ------------------
function HomeScreen({ navigation }) {
  return (
    <SafeAreaView style={styles.container}>
      <ImageBackground
        source={require('./assets/Background.png')}
        style={styles.background}
        resizeMode="cover"
      >
        <View style={styles.logoContainer}>
          <Image source={require('./assets/TasteTap.webp')} style={styles.logo} resizeMode="contain" />
          <Text style={styles.logoText}>Taste Tap</Text>
        </View>

       
        <Text style={styles.h2}>Welcome to Taste Tap</Text>
       
        {/* Proceed goes straight to Menu */}
        <TouchableOpacity style={styles.mainButton} onPress={() => navigation.navigate('Menu')}>
          <Text style={styles.buttonText}>Proceed</Text>
        </TouchableOpacity>

        <TouchableOpacity style={[styles.mainButton, {backgroundColor:'#333'}]} onPress={() => navigation.navigate('AdminLogin')}>
          <Text style={styles.buttonText}>Admin Login</Text>
        </TouchableOpacity>

        <Text style={styles.slogan}>Savor Every Sip, Relish Every Bite</Text>
      </ImageBackground>
    </SafeAreaView>
  );
}

// ------------------ ADMIN LOGIN ------------------
function AdminLoginScreen({ navigation }) {
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = () => {
    if(username === 'Admin' && password === '12345') {
      navigation.navigate('AdminDashboard');
    } else {
      Alert.alert('Error', 'Invalid credentials');
    }
  };

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.h1}>Admin Login</Text>
      <TextInput style={styles.input} placeholder="Username" value={username} onChangeText={setUsername} />
      <TextInput style={styles.input} placeholder="Password" value={password} onChangeText={setPassword} secureTextEntry />
      <TouchableOpacity style={styles.mainButton} onPress={handleLogin}>
        <Text style={styles.buttonText}>Login</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// ------------------ ADMIN DASHBOARD ------------------
function AdminDashboard({ navigation }) {
  const { mainMenu, setMainMenu, dessertMenu, setDessertMenu, specialMenu, setSpecialMenu } = useContext(MenuContext);
  const [selectedMenu, setSelectedMenu] = useState('Main');
  const [newItem, setNewItem] = useState('');
  const [newPrice, setNewPrice] = useState('');

  const getCurrentMenu = () => {
    if(selectedMenu === 'Main') return [mainMenu, setMainMenu];
    if(selectedMenu === 'Dessert') return [dessertMenu, setDessertMenu];
    if(selectedMenu === 'Special') return [specialMenu, setSpecialMenu];
  }

  const addMenuItem = () => {
    if(newItem && newPrice) {
      const [menu, setMenu] = getCurrentMenu();
      setMenu([...menu, { id: Date.now(), title: newItem, price: parseFloat(newPrice) }]);
      setNewItem('');
      setNewPrice('');
    }
  }

  const removeItem = (id) => {
    const [menu, setMenu] = getCurrentMenu();
    setMenu(menu.filter(item => item.id !== id));
  }

  return (
    <SafeAreaView style={styles.container}>
      {/* Logout button (top-right) */}
      <TouchableOpacity style={[styles.cardButton, styles.logout]} onPress={() => navigation.navigate('Home')}>
        <Text style={styles.buttonText}>Logout</Text>
      </TouchableOpacity>

      <Text style={styles.h1}>Admin Dashboard</Text>

      <View style={{flexDirection:'row', justifyContent:'space-around', marginVertical:10}}>
        {['Main','Dessert','Special'].map(menu => (
          <TouchableOpacity key={menu} style={[styles.secondaryButton, selectedMenu===menu && {backgroundColor:'#FF0000'}]} onPress={()=>setSelectedMenu(menu)}>
            <Text style={styles.buttonText}>{menu} Menu</Text>
          </TouchableOpacity>
        ))}
      </View>

      <TextInput style={styles.input} placeholder="New Item Name" value={newItem} onChangeText={setNewItem} />
      <TextInput style={styles.input} placeholder="Price" value={newPrice} onChangeText={setNewPrice} keyboardType="numeric" />
      <TouchableOpacity style={styles.mainButton} onPress={addMenuItem}>
        <Text style={styles.buttonText}>Add Item</Text>
      </TouchableOpacity>

      <ScrollView style={{marginVertical:10}}>
        {getCurrentMenu()[0].map(item => (
          <View key={item.id} style={{flexDirection:'row', justifyContent:'space-between', padding:10, borderBottomWidth:1, borderColor:'#ccc'}}>
            <Text>{item.title} - R{item.price}</Text>
            <TouchableOpacity style={[styles.cardButton, {padding:5}]} onPress={()=>removeItem(item.id)}>
              <Text style={styles.buttonText}>Remove</Text>
            </TouchableOpacity>
          </View>
        ))}
      </ScrollView>

      <TouchableOpacity style={[styles.mainButton, {backgroundColor:'#333'}]} onPress={() => navigation.navigate('Menu')}>
        <Text style={styles.buttonText}>Go to User Side</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// ------------------ DETAILS PAGE ------------------
function DetailsScreen({ navigation }) {
  const [name, setName] = useState('');
  const [surname, setSurname] = useState('');

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.h1}>Enter Your Details</Text>
      <TextInput style={styles.input} placeholder="Enter your name" value={name} onChangeText={setName} />
      <TextInput style={styles.input} placeholder="Enter your surname" value={surname} onChangeText={setSurname} />
      {/* Continue to Checkout */}
      <TouchableOpacity style={styles.mainButton} onPress={() => navigation.navigate('Checkout', { name, surname })}>
        <Text style={styles.buttonText}>Continue to Checkout</Text>
      </TouchableOpacity>
    </SafeAreaView>
  );
}

// ------------------ MENU PAGE ------------------
function MenuScreen({ navigation }) {
  const { cartItems, addToCart } = useContext(CartContext);
  const { mainMenu, dessertMenu, specialMenu } = useContext(MenuContext);

  const [currentMenu, setCurrentMenu] = useState('Main');
  const [fadeAnim] = useState(new Animated.Value(0));
  const [cartScale] = useState(new Animated.Value(1));

  const animateCards = () => {
    Animated.timing(fadeAnim, { toValue: 1, duration: 800, useNativeDriver: true }).start();
  };

  const animateCart = () => {
    Animated.sequence([
      Animated.timing(cartScale, { toValue: 1.3, duration: 150, useNativeDriver: true }),
      Animated.timing(cartScale, { toValue: 1, duration: 150, useNativeDriver: true }),
    ]).start();
  };

  useEffect(()=>{ animateCards() }, [currentMenu]);

  const getMenuItems = () => {
    if(currentMenu==='Main') return mainMenu;
    if(currentMenu==='Dessert') return dessertMenu;
    if(currentMenu==='Special') return specialMenu;
  }

  const handleAddToCart = (item) => { addToCart(item); animateCart(); }

  const handleCashOut = () => {
    if (cartItems.length === 0) {
      Alert.alert('Cart Empty', 'Please add at least one item.');
      return;
    }
    Alert.alert('Checkout', `You have ${cartItems.length} items in your cart.`, [
      { text:'OK', onPress:()=>navigation.navigate('Details') }
    ]);
  };

  return (
    <SafeAreaView style={styles.container}>
      {/* Logout Button (top-right) */}
      <TouchableOpacity style={[styles.cardButton, styles.logout]} onPress={() => navigation.navigate('Home')}>
        <Text style={styles.buttonText}>Logout</Text>
      </TouchableOpacity>

      <ScrollView>
        <Text style={styles.h1}>Menu</Text>
        <Text style={styles.h2}>What would you like to order?</Text>

        <View style={{flexDirection:'row', justifyContent:'space-around', marginVertical:10}}>
          {['Main','Dessert','Special'].map(menu => (
            <TouchableOpacity key={menu} style={[styles.secondaryButton, currentMenu===menu && {backgroundColor:'#FF0000'}]} onPress={()=>setCurrentMenu(menu)}>
              <Text style={styles.buttonText}>{menu} Menu</Text>
            </TouchableOpacity>
          ))}
        </View>

        <View style={styles.grid}>
          {getMenuItems().map(item => (
            <Animated.View key={item.id} style={[styles.card, {opacity: fadeAnim}]}>
              <Text style={styles.foodTitle}>{item.title}</Text>
              <Text style={styles.price}>R{item.price}</Text>
              <TouchableOpacity style={styles.cardButton} onPress={()=>handleAddToCart(item)}>
                <Text style={styles.buttonText}>Add to Cart</Text>
              </TouchableOpacity>
            </Animated.View>
          ))}
        </View>

        <View style={styles.cashOutButton}>
          <TouchableOpacity style={styles.mainButton} onPress={handleCashOut}>
            <Text style={styles.buttonText}>Cash Out</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>

      <View style={styles.bottomBar}>
        <Animated.Text style={[styles.cartText, { transform:[{scale:cartScale}] }]}>
          🛒 Cart: {cartItems.length}
        </Animated.Text>
      </View>
    </SafeAreaView>
  );
}

// ------------------ CHECKOUT PAGE ------------------
function CheckoutScreen({ route }) {
  const { cartItems, clearCart } = useContext(CartContext);
  const total = cartItems.reduce((sum, item)=>sum+item.price,0);
  const name = route.params?.name;
  const surname = route.params?.surname;

  const handlePlaceOrder = () => {
    const orderNumber = Math.floor(Math.random()*1000000);
    Alert.alert('Order Placed', `Your order number is #${orderNumber}. Please pay at the till.`, [{ text:'OK', onPress:clearCart }]);
  }

  return (
    <SafeAreaView style={styles.container}>
      <Text style={styles.h1}>Order Summary</Text>
      {name || surname ? (
        <Text style={styles.p1}>For: {name} {surname}</Text>
      ) : null}
      <ScrollView>
        {cartItems.map((item,index)=>(
          <View key={index} style={{marginVertical:5}}>
            <Text>{item.title} - R{item.price}</Text>
          </View>
        ))}
        <Text style={{marginTop:10, fontWeight:'bold', fontSize:18}}>Total: R{total}</Text>
        <View style={{marginTop:20}}>
          <TouchableOpacity style={styles.mainButton} onPress={handlePlaceOrder}>
            <Text style={styles.buttonText}>Place Order</Text>
          </TouchableOpacity>
        </View>
      </ScrollView>
    </SafeAreaView>
  );
}

// ------------------ APP ------------------
export default function App() {
  const [cartItems, setCartItems] = useState([]);
  const [mainMenu, setMainMenu] = useState([
    { id: 1, title: 'Burger', price: 100 },
    { id: 2, title: 'Salad', price: 80 },
    { id: 3, title: 'Pizza', price: 120 },
    { id: 4, title: 'Pasta', price: 110 },
    { id: 5, title: 'Dessert', price: 60 },
    { id: 6, title: 'Drink', price: 40 },
  ]);
  const [dessertMenu, setDessertMenu] = useState([
    { id: 1, title: 'Ice Cream', price: 50 },
    { id: 2, title: 'Donut', price: 40 },
    { id: 3, title: 'Cookie', price: 30 },
  ]);
  const [specialMenu, setSpecialMenu] = useState([
    { id: 1, title: 'Steak', price: 200 },
    { id: 2, title: 'Sushi', price: 180 },
  ]);

  const addToCart = (item) => setCartItems([...cartItems,item]);
  const clearCart = () => setCartItems([]);

  return (
    <MenuContext.Provider value={{ mainMenu, setMainMenu, dessertMenu, setDessertMenu, specialMenu, setSpecialMenu }}>
      <CartContext.Provider value={{ cartItems, addToCart, clearCart }}>
        <NavigationContainer>
          <Stack.Navigator initialRouteName="Home">
            <Stack.Screen name="Home" component={HomeScreen} />
            <Stack.Screen name="AdminLogin" component={AdminLoginScreen} />
            <Stack.Screen name="AdminDashboard" component={AdminDashboard} />
            <Stack.Screen name="Details" component={DetailsScreen} />
            <Stack.Screen name="Menu" component={MenuScreen} />
            <Stack.Screen name="Checkout" component={CheckoutScreen} />
          </Stack.Navigator>
        </NavigationContainer>
      </CartContext.Provider>
    </MenuContext.Provider>
  );
}

// ------------------ STYLES ------------------
const styles = StyleSheet.create({
  container: { flex:1, padding:8, backgroundColor:'#FDFDFD' },
  background: { flex:1 },
  logoContainer: { flexDirection:'row', alignItems:'center', margin:20 },
  logo: { width:60, height:60, marginRight:10 },
  logoText: { fontSize:40, fontWeight:'bold', color:'#FF0000' },
  h1: { margin:10, fontSize:25, fontWeight:'bold', color:'#FF0000' },
  h2: { margin:10, fontSize:30, fontWeight:'bold', color:'#FF0000' },
  p1: { margin:10, fontSize:18, color:'black' },
  slogan: { margin:10, fontSize:20, fontWeight:'bold', color:'black', fontFamily: 'Snell Roundhand' },
  input: { borderWidth:1, borderColor:'#ccc', padding:12, marginBottom:15, borderRadius:8, fontSize:16 },
  mainButton: { backgroundColor:'#FF0000', padding:12, borderRadius:8, alignItems:'center', marginVertical:8 },
  secondaryButton: { backgroundColor:'#FFA500', padding:12, borderRadius:8, alignItems:'center', flex:0.3 },
  cardButton: { backgroundColor:'#FF4500', padding:8, borderRadius:6, alignItems:'center', marginTop:5 },
  buttonText: { color:'#fff', fontWeight:'bold', fontSize:16 },
  grid: { flexDirection:'row', flexWrap:'wrap', justifyContent:'space-between' },
  card: { backgroundColor:'#fff', padding:15, marginVertical:10, borderRadius:12, shadowColor:'#000', shadowOpacity:0.15, shadowOffset:{width:0,height:4}, shadowRadius:6, elevation:6, width:'48%' },
  foodTitle: { fontSize:18, fontWeight:'bold', marginBottom:5 },
  price: { fontSize:16, fontWeight:'600', marginVertical:5, color:'#FF0000' },
  cashOutButton: { marginVertical:15 },
  bottomBar: { flexDirection:'row', justifyContent:'space-between', alignItems:'center', padding:12, borderTopWidth:1, borderColor:'#ddd', backgroundColor:'#f7f7f7' },
  cartText: { fontSize:18, fontWeight:'bold', color:'#FF0000' },
  logout: { position:'absolute', top:10, right:10, zIndex:10 }
});
