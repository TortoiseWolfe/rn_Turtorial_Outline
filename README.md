# Building a Secure Steampunk-Themed React Native App

## Initial Setup

```bash
npx create-expo-app@latest steampunk-app
cd steampunk-app
npx expo install expo-router react-native-safe-area-context react-native-screens expo-linking expo-constants expo-status-bar expo-font
npx expo install @supabase/supabase-js
npx expo install formik yup
npx expo install nativewind tailwindcss@3.3.2
npx expo install @react-native-async-storage/async-storage
```

## Project Structure

```bash
mkdir -p app/screens app/components app/hooks app/utils app/constants
touch app.config.js tailwind.config.js babel.config.js
touch app/hooks/useAuth.js app/utils/supabase.js
touch .env .env.example
```

## Environment Configuration

`.env.example`:
```plaintext
EXPO_PUBLIC_SUPABASE_URL=your_supabase_url
EXPO_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

`.env`:
```plaintext
EXPO_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
EXPO_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

## Supabase Configuration

`app/utils/supabase.js`:
```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';
import { createClient } from '@supabase/supabase-js';
import { EXPO_PUBLIC_SUPABASE_URL, EXPO_PUBLIC_SUPABASE_ANON_KEY } from '@env';

const supabaseUrl = EXPO_PUBLIC_SUPABASE_URL;
const supabaseAnonKey = EXPO_PUBLIC_SUPABASE_ANON_KEY;

export const supabase = createClient(supabaseUrl, supabaseAnonKey, {
  auth: {
    storage: AsyncStorage,
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: false,
  },
});
```

## NativeWind Configuration

`tailwind.config.js`:
```javascript
module.exports = {
  content: ["./app/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {
      colors: {
        steampunk: {
          brass: '#B5A642',
          copper: '#B87333',
          bronze: '#CD7F32',
          leather: '#8B4513',
          wood: '#DEB887',
        },
      },
    },
  },
  plugins: [],
};
```

## Authentication Hook

`app/hooks/useAuth.js`:
```javascript
import { create } from 'zustand';
import { supabase } from '../utils/supabase';

const useAuth = create((set) => ({
  user: null,
  session: null,
  loading: true,
  signUp: async (email, password) => {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
    });
    if (error) throw error;
    return data;
  },
  signIn: async (email, password) => {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });
    if (error) throw error;
    set({ user: data.user, session: data.session });
    return data;
  },
  signOut: async () => {
    await supabase.auth.signOut();
    set({ user: null, session: null });
  },
}));

export default useAuth;
```

## Router Configuration

`app/_layout.js`:
```javascript
import { Stack } from 'expo-router';
import { useEffect } from 'react';
import { useFonts } from 'expo-font';
import useAuth from '../hooks/useAuth';

export default function Layout() {
  const [fontsLoaded] = useFonts({
    'SpecialElite': require('../assets/fonts/SpecialElite-Regular.ttf'),
    'ArbutusSlab': require('../assets/fonts/ArbutusSlab-Regular.ttf'),
  });

  const { user } = useAuth();

  if (!fontsLoaded) return null;

  return (
    <Stack screenOptions={{
      headerStyle: {
        backgroundColor: '#B5A642',
      },
      headerTintColor: '#fff',
      headerTitleStyle: {
        fontFamily: 'SpecialElite',
      },
    }}>
      <Stack.Screen
        name="(auth)/login"
        options={{
          headerShown: false,
          href: !user ? '/login' : null,
        }}
      />
      <Stack.Screen
        name="(auth)/signup"
        options={{
          headerShown: false,
          href: !user ? '/signup' : null,
        }}
      />
      <Stack.Screen
        name="(app)/profile"
        options={{
          href: user ? '/profile' : null,
        }}
      />
    </Stack>
  );
}
```

## Signup Form

`app/(auth)/signup.js`:
```javascript
import { View, Text, TextInput, TouchableOpacity } from 'react-native';
import { Formik } from 'formik';
import * as Yup from 'yup';
import useAuth from '../../hooks/useAuth';

const SignupSchema = Yup.object().shape({
  email: Yup.string().email('Invalid email').required('Required'),
  password: Yup.string()
    .min(8, 'Too Short!')
    .required('Required'),
});

export default function SignUp() {
  const { signUp } = useAuth();

  return (
    <View className="flex-1 bg-steampunk-wood p-4">
      <Formik
        initialValues={{ email: '', password: '' }}
        validationSchema={SignupSchema}
        onSubmit={async (values) => {
          try {
            await signUp(values.email, values.password);
          } catch (error) {
            alert(error.message);
          }
        }}
      >
        {({ handleChange, handleSubmit, values, errors, touched }) => (
          <View className="space-y-4">
            <TextInput
              className="bg-white p-2 rounded font-['ArbutusSlab']"
              placeholder="Email"
              onChangeText={handleChange('email')}
              value={values.email}
            />
            {errors.email && touched.email && (
              <Text className="text-red-500">{errors.email}</Text>
            )}
            <TextInput
              className="bg-white p-2 rounded font-['ArbutusSlab']"
              placeholder="Password"
              secureTextEntry
              onChangeText={handleChange('password')}
              value={values.password}
            />
            {errors.password && touched.password && (
              <Text className="text-red-500">{errors.password}</Text>
            )}
            <TouchableOpacity
              className="bg-steampunk-brass p-4 rounded"
              onPress={handleSubmit}
            >
              <Text className="text-white text-center font-['SpecialElite']">
                Sign Up
              </Text>
            </TouchableOpacity>
          </View>
        )}
      </Formik>
    </View>
  );
}
```

## Profile Page

`app/(app)/profile.js`:
```javascript
import { View, Text, TouchableOpacity } from 'react-native';
import useAuth from '../../hooks/useAuth';

export default function Profile() {
  const { user, signOut } = useAuth();

  return (
    <View className="flex-1 bg-steampunk-wood p-4">
      <View className="bg-steampunk-bronze p-6 rounded-lg">
        <Text className="font-['SpecialElite'] text-white text-xl mb-2">
          Profile Information
        </Text>
        <Text className="font-['ArbutusSlab'] text-white mb-4">
          Email: {user?.email}
        </Text>
        <TouchableOpacity
          className="bg-steampunk-leather p-4 rounded"
          onPress={signOut}
        >
          <Text className="text-white text-center font-['SpecialElite']">
            Sign Out
          </Text>
        </TouchableOpacity>
      </View>
    </View>
  );
}
```

## Running the Application

1. Download the required fonts and place them in `assets/fonts/`
2. Configure Supabase:
   - Create a new project
   - Enable Email auth
   - Copy URL and anon key to `.env`

```bash
npx expo start
```

## Testing

1. Sign up with a new account
2. Verify protected routes work
3. Test dark/light theme
4. Verify form validation
5. Test authentication flow

## Security Notes

- Environment variables are exposed to the client in Expo
- Use Row Level Security in Supabase
- Implement proper session handling
- Add rate limiting for auth endpoints

