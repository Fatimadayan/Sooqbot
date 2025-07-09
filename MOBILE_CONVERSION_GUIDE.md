# Mobile App Conversion Guide - SooqBot

## Overview
This guide shows you how to convert your web-based SooqBot e-commerce platform into a mobile application using React Native with Expo.

## Option 1: React Native with Expo (Recommended)

### Prerequisites
```bash
npm install -g @expo/cli
npx create-expo-app SooqBotMobile --template blank-typescript
cd SooqBotMobile
```

### Key Dependencies for Mobile
```bash
npx expo install expo-router @react-navigation/native @react-navigation/stack
npx expo install react-native-screens react-native-safe-area-context
npx expo install @tanstack/react-query expo-secure-store
npx expo install expo-image-picker expo-camera
npx expo install react-native-paper # Alternative to shadcn for mobile
```

### Folder Structure
```
SooqBotMobile/
├── app/                    # Expo Router pages
│   ├── (tabs)/            # Tab navigation
│   │   ├── dashboard.tsx
│   │   ├── stores.tsx
│   │   └── profile.tsx
│   ├── create-store.tsx
│   ├── store/[id].tsx     # Dynamic store pages
│   └── _layout.tsx
├── components/            # Reusable components
├── hooks/                 # Custom hooks
├── services/             # API services
├── types/                # TypeScript types
└── constants/            # App constants
```

## Component Conversion Examples

### 1. Store Creation Form (Web → Mobile)

**Web Version (client/src/pages/create-store.tsx):**
```tsx
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";

export default function CreateStore() {
  return (
    <Card className="max-w-md mx-auto">
      <CardContent>
        <form onSubmit={handleSubmit}>
          <input placeholder="Store Name" />
          <Button>Create Store</Button>
        </form>
      </CardContent>
    </Card>
  );
}
```

**Mobile Version (app/create-store.tsx):**
```tsx
import { View, ScrollView } from 'react-native';
import { Card, TextInput, Button } from 'react-native-paper';
import { SafeAreaView } from 'react-native-safe-area-context';

export default function CreateStore() {
  return (
    <SafeAreaView style={{ flex: 1 }}>
      <ScrollView contentContainerStyle={{ padding: 16 }}>
        <Card style={{ marginBottom: 16 }}>
          <Card.Content>
            <TextInput
              label="Store Name"
              mode="outlined"
              value={storeName}
              onChangeText={setStoreName}
            />
            <Button mode="contained" onPress={handleSubmit}>
              Create Store
            </Button>
          </Card.Content>
        </Card>
      </ScrollView>
    </SafeAreaView>
  );
}
```

### 2. Navigation (Web → Mobile)

**Web Version (client/src/App.tsx):**
```tsx
import { Route, Switch } from 'wouter';

function App() {
  return (
    <Switch>
      <Route path="/" component={Home} />
      <Route path="/create-store" component={CreateStore} />
      <Route path="/dashboard" component={Dashboard} />
    </Switch>
  );
}
```

**Mobile Version (app/_layout.tsx):**
```tsx
import { Stack } from 'expo-router';

export default function RootLayout() {
  return (
    <Stack>
      <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
      <Stack.Screen name="create-store" options={{ title: 'Create Store' }} />
      <Stack.Screen name="store/[id]" options={{ title: 'Store Details' }} />
    </Stack>
  );
}
```

### 3. API Integration (Shared Logic)

**Create shared API service (services/api.ts):**
```tsx
import { QueryClient } from '@tanstack/react-query';

const API_BASE = __DEV__ 
  ? 'http://localhost:3001' 
  : 'https://your-production-url.com';

export const api = {
  createStore: async (data: StoreData) => {
    const response = await fetch(`${API_BASE}/api/stores`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data),
    });
    return response.json();
  },
  
  getStores: async () => {
    const response = await fetch(`${API_BASE}/api/stores`);
    return response.json();
  }
};
```

## Mobile-Specific Features

### 1. Image Picker for Product Photos
```tsx
import * as ImagePicker from 'expo-image-picker';

const pickImage = async () => {
  const result = await ImagePicker.launchImageLibraryAsync({
    mediaTypes: ImagePicker.MediaTypeOptions.Images,
    allowsEditing: true,
    aspect: [1, 1],
    quality: 0.8,
  });

  if (!result.canceled) {
    setProductImage(result.assets[0].uri);
  }
};
```

### 2. Push Notifications for Orders
```tsx
import * as Notifications from 'expo-notifications';

const scheduleOrderNotification = async (orderData: Order) => {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: 'New Order Received!',
      body: `Order #${orderData.id} for ${orderData.totalAmount} SAR`,
    },
    trigger: null, // Show immediately
  });
};
```

### 3. Offline Support with AsyncStorage
```tsx
import AsyncStorage from '@react-native-async-storage/async-storage';

const saveOfflineData = async (data: any) => {
  try {
    await AsyncStorage.setItem('offline_stores', JSON.stringify(data));
  } catch (error) {
    console.error('Failed to save offline data:', error);
  }
};
```

## Styling Differences

### Web (Tailwind CSS)
```tsx
<div className="bg-white rounded-lg shadow-md p-4">
  <h2 className="text-xl font-bold mb-4">Store Dashboard</h2>
</div>
```

### Mobile (StyleSheet)
```tsx
import { StyleSheet } from 'react-native';

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#ffffff',
    borderRadius: 8,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.1,
    shadowRadius: 4,
    elevation: 3,
    padding: 16,
  },
  title: {
    fontSize: 20,
    fontWeight: 'bold',
    marginBottom: 16,
  },
});

<View style={styles.container}>
  <Text style={styles.title}>Store Dashboard</Text>
</View>
```

## Arabic RTL Support in Mobile

### app.json Configuration
```json
{
  "expo": {
    "name": "SooqBot",
    "locales": {
      "ar": "./locales/ar.json"
    },
    "ios": {
      "supportsTablet": true
    },
    "android": {
      "allowBackup": false
    }
  }
}
```

### RTL Text Support
```tsx
import { I18nManager } from 'react-native';

// Enable RTL
I18nManager.forceRTL(true);

// In component
<Text style={{ textAlign: I18nManager.isRTL ? 'right' : 'left' }}>
  مرحبا بكم في سوق بوت
</Text>
```

## Build and Deployment

### Development Build
```bash
npx expo run:ios
npx expo run:android
```

### Production Build
```bash
# For App Store
eas build --platform ios

# For Google Play Store
eas build --platform android
```

### App Store Configuration
```json
// app.json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.yourcompany.sooqbot",
      "buildNumber": "1.0.0"
    },
    "android": {
      "package": "com.yourcompany.sooqbot",
      "versionCode": 1
    }
  }
}
```

## Testing Strategy

### Unit Tests
```bash
npm install --save-dev jest @testing-library/react-native
```

### Device Testing
```bash
# iOS Simulator
npx expo run:ios

# Android Emulator
npx expo run:android

# Physical Device
npx expo start --tunnel
```

## Performance Optimizations

### 1. Lazy Loading
```tsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));

<Suspense fallback={<ActivityIndicator />}>
  <Dashboard />
</Suspense>
```

### 2. Image Optimization
```tsx
import { Image } from 'expo-image';

<Image
  source={{ uri: productImage }}
  style={{ width: 200, height: 200 }}
  contentFit="cover"
  cachePolicy="memory-disk"
/>
```

### 3. Memory Management
```tsx
import { useFocusEffect } from '@react-navigation/native';

useFocusEffect(
  useCallback(() => {
    // Load data when screen is focused
    loadStoreData();
    
    return () => {
      // Cleanup when screen loses focus
      clearStoreData();
    };
  }, [])
);
```

## Next Steps

1. **Set up Expo project** with the commands above
2. **Copy shared types** from `shared/schema.ts`
3. **Convert core components** starting with store creation
4. **Implement navigation** using Expo Router
5. **Add mobile-specific features** like image picker
6. **Test on both iOS and Android**
7. **Deploy to app stores** using EAS Build

This conversion will give you a native mobile experience while reusing most of your business logic and API integration.