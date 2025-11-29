# Evergreen Home - Development Roadmap

## Project Overview
**App Name:** Evergreen Home  
**Purpose:** Affordable housing finder for Seattle/Washington State with intelligent income-based eligibility filtering  
**Tech Stack:** React Native + Expo + NativeWind  
**Target Platform:** iOS and Android (App Store & Google Play)  
**Development Timeline:** 2-4 months for MVP

---

## Core Value Proposition
A mobile-first affordable housing finder that helps Seattle-area residents quickly find rental units they actually qualify for by:
1. Calculating eligibility based on household income and size
2. Aggregating listings from multiple affordable housing sources
3. Filtering to show only qualified units with clear eligibility indicators
4. Integrating transit information for location-based decisions

---

## Phase 1: Project Setup & Foundation (Week 1-2)

### 1.1 Initialize Project
```bash
npx create-expo-app evergreen-home --template blank
cd evergreen-home
npx expo install nativewind
npx expo install tailwindcss
```

### 1.2 Install Core Dependencies
```bash
# Navigation
npx expo install @react-navigation/native @react-navigation/native-stack @react-navigation/bottom-tabs
npx expo install react-native-screens react-native-safe-area-context

# Maps & Location
npx expo install react-native-maps expo-location

# Storage
npx expo install @react-native-async-storage/async-storage

# Icons
npx expo install @expo/vector-icons

# UI Components
npx expo install react-native-paper
```

### 1.3 Configure NativeWind
Create `tailwind.config.js`:
```javascript
module.exports = {
  content: [
    "./App.{js,jsx,ts,tsx}",
    "./src/**/*.{js,jsx,ts,tsx}"
  ],
  theme: {
    extend: {
      colors: {
        primary: '#2D5F3F', // Evergreen
        secondary: '#4A7C59',
        accent: '#F59E0B',
      }
    },
  },
  plugins: [],
}
```

### 1.4 Project Structure
```
evergreen-home/
├── src/
│   ├── components/
│   │   ├── common/           # Reusable UI components
│   │   ├── listings/         # Listing-specific components
│   │   └── map/              # Map-related components
│   ├── screens/
│   │   ├── OnboardingScreen.js
│   │   ├── HomeScreen.js
│   │   ├── MapScreen.js
│   │   ├── ListingDetailScreen.js
│   │   └── SavedScreen.js
│   ├── navigation/
│   │   └── AppNavigator.js
│   ├── utils/
│   │   ├── eligibility.js    # AMI calculations
│   │   ├── storage.js        # AsyncStorage helpers
│   │   └── constants.js      # App-wide constants
│   ├── data/
│   │   └── listings.json     # Initial mock data
│   └── hooks/
│       ├── useEligibility.js
│       └── useListings.js
├── assets/
│   ├── images/
│   └── icons/
├── app.json
├── package.json
└── ROADMAP.md
```

### 1.5 App Configuration (app.json)
```json
{
  "expo": {
    "name": "Evergreen Home",
    "slug": "evergreen-home",
    "version": "1.0.0",
    "orientation": "portrait",
    "icon": "./assets/icon.png",
    "userInterfaceStyle": "automatic",
    "splash": {
      "image": "./assets/splash.png",
      "resizeMode": "contain",
      "backgroundColor": "#2D5F3F"
    },
    "ios": {
      "supportsTablet": true,
      "bundleIdentifier": "com.evergreenhome.app"
    },
    "android": {
      "adaptiveIcon": {
        "foregroundImage": "./assets/adaptive-icon.png",
        "backgroundColor": "#2D5F3F"
      },
      "package": "com.evergreenhome.app",
      "permissions": ["ACCESS_FINE_LOCATION"]
    },
    "plugins": [
      [
        "expo-location",
        {
          "locationAlwaysAndWhenInUsePermission": "Allow Evergreen Home to use your location to show nearby affordable housing."
        }
      ]
    ]
  }
}
```

---

## Phase 2: Core Data Structure & Logic (Week 2-3)

### 2.1 Define Listing Data Schema
Create `src/types/listing.ts` (or .js with JSDoc):
```javascript
/**
 * @typedef {Object} Listing
 * @property {string} id - Unique identifier
 * @property {string} name - Property name
 * @property {string} address - Full address
 * @property {number} lat - Latitude
 * @property {number} lng - Longitude
 * @property {number} rent - Monthly rent
 * @property {number} bedrooms - Number of bedrooms
 * @property {number} bathrooms - Number of bathrooms
 * @property {number} sqft - Square footage
 * @property {string[]} images - Array of image URLs
 * @property {string} program - Program type (MFTE, MHA, Section8, SHA, LIHTC)
 * @property {number} amiPercentage - AMI percentage (30, 50, 60, 80, 120)
 * @property {Object} incomeLimit - Income limits by household size
 * @property {string} contactPhone - Property manager phone
 * @property {string} contactEmail - Property manager email
 * @property {string} websiteUrl - Application website
 * @property {string} managementCompany - Property management company
 * @property {boolean} available - Current availability status
 * @property {string} neighborhood - Seattle neighborhood
 * @property {string[]} amenities - List of amenities
 * @property {string} description - Property description
 */
```

### 2.2 AMI Calculation Utility
Create `src/utils/eligibility.js`:
```javascript
// 2025 Seattle Area Median Income (AMI) data
// Source: HUD/Seattle Office of Housing
const AMI_2025 = {
  1: 93800,  // 1 person household
  2: 107200,
  3: 120600,
  4: 133900,
  5: 144700,
  6: 155500,
  7: 166200,
  8: 177000,
};

export const calculateMaxIncome = (householdSize, amiPercentage) => {
  const baseAMI = AMI_2025[householdSize] || AMI_2025[8];
  return Math.round(baseAMI * (amiPercentage / 100));
};

export const checkEligibility = (userIncome, householdSize, listingAMI) => {
  const maxIncome = calculateMaxIncome(householdSize, listingAMI);
  return userIncome <= maxIncome;
};

export const getUserAMIPercentage = (userIncome, householdSize) => {
  const baseAMI = AMI_2025[householdSize] || AMI_2025[8];
  return Math.round((userIncome / baseAMI) * 100);
};
```

### 2.3 Storage Utility
Create `src/utils/storage.js`:
```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';

const KEYS = {
  USER_INCOME: '@evergreen_user_income',
  HOUSEHOLD_SIZE: '@evergreen_household_size',
  SAVED_LISTINGS: '@evergreen_saved_listings',
  ONBOARDING_COMPLETE: '@evergreen_onboarding',
};

export const saveUserProfile = async (income, householdSize) => {
  try {
    await AsyncStorage.multiSet([
      [KEYS.USER_INCOME, income.toString()],
      [KEYS.HOUSEHOLD_SIZE, householdSize.toString()],
      [KEYS.ONBOARDING_COMPLETE, 'true'],
    ]);
  } catch (error) {
    console.error('Error saving user profile:', error);
  }
};

export const getUserProfile = async () => {
  try {
    const values = await AsyncStorage.multiGet([
      KEYS.USER_INCOME,
      KEYS.HOUSEHOLD_SIZE,
    ]);
    return {
      income: parseInt(values[0][1]) || null,
      householdSize: parseInt(values[1][1]) || null,
    };
  } catch (error) {
    console.error('Error getting user profile:', error);
    return { income: null, householdSize: null };
  }
};

export const toggleSavedListing = async (listingId) => {
  try {
    const saved = await AsyncStorage.getItem(KEYS.SAVED_LISTINGS);
    const savedArray = saved ? JSON.parse(saved) : [];
    
    const index = savedArray.indexOf(listingId);
    if (index > -1) {
      savedArray.splice(index, 1);
    } else {
      savedArray.push(listingId);
    }
    
    await AsyncStorage.setItem(KEYS.SAVED_LISTINGS, JSON.stringify(savedArray));
    return savedArray;
  } catch (error) {
    console.error('Error toggling saved listing:', error);
  }
};

export const getSavedListings = async () => {
  try {
    const saved = await AsyncStorage.getItem(KEYS.SAVED_LISTINGS);
    return saved ? JSON.parse(saved) : [];
  } catch (error) {
    console.error('Error getting saved listings:', error);
    return [];
  }
};
```

### 2.4 Create Mock Data
Create `src/data/listings.json` with 10-20 sample listings:
```json
[
  {
    "id": "1",
    "name": "Greenwood Plaza Apartments",
    "address": "8515 Greenwood Ave N, Seattle, WA 98103",
    "lat": 47.6911,
    "lng": -122.3542,
    "rent": 1200,
    "bedrooms": 1,
    "bathrooms": 1,
    "sqft": 650,
    "images": ["https://example.com/image1.jpg"],
    "program": "MFTE",
    "amiPercentage": 80,
    "contactPhone": "(206) 555-0123",
    "contactEmail": "leasing@greenwoodplaza.com",
    "websiteUrl": "https://greenwoodplaza.com/apply",
    "managementCompany": "Seattle Property Management",
    "available": true,
    "neighborhood": "Greenwood",
    "amenities": ["Laundry", "Parking", "Pet Friendly"],
    "description": "Beautiful one-bedroom apartment in Greenwood..."
  }
]
```

---

## Phase 3: UI Components Development (Week 3-5)

### 3.1 Onboarding Screen
**Features:**
- Welcome message explaining the app
- Income input field (validated, numeric only)
- Household size selector (1-8+)
- "Get Started" button that saves profile and navigates to home

**Components to build:**
- `OnboardingScreen.js`
- `IncomeInput.js` (custom component with formatting)
- `HouseholdSizeSelector.js` (button group or picker)

### 3.2 Home Screen (List View)
**Features:**
- Header with user's AMI percentage display
- Search bar (filter by name/address)
- Filter controls (bedrooms, rent range, program type, neighborhood)
- Listing cards showing:
  - Property image
  - Name and address
  - Rent, bed/bath
  - Eligibility badge (green checkmark or warning icon)
  - Heart icon for saving
- Pull-to-refresh functionality

**Components to build:**
- `HomeScreen.js`
- `ListingCard.js`
- `FilterBar.js`
- `EligibilityBadge.js`
- `SearchBar.js`

### 3.3 Map Screen
**Features:**
- Full-screen map with listing pins
- Cluster markers when zoomed out
- Custom pin colors (green = eligible, gray = not eligible)
- Bottom sheet showing listing preview on pin tap
- Toggle to switch to list view
- "My Location" button

**Components to build:**
- `MapScreen.js`
- `CustomMarker.js`
- `ListingPreviewSheet.js`

### 3.4 Listing Detail Screen
**Features:**
- Image carousel
- Full property details
- Eligibility section with explanation
- "Your income qualifies!" or "Income too high" message
- Transit information section (nearest stops)
- Amenities list
- Contact information
- "Apply Now" button (opens phone/email/web)
- Save/unsave button

**Components to build:**
- `ListingDetailScreen.js`
- `ImageCarousel.js`
- `EligibilityCard.js`
- `TransitInfo.js`
- `ContactButtons.js`

### 3.5 Saved Listings Screen
**Features:**
- List of saved listings (same ListingCard component)
- Empty state when no saved listings
- Remove from saved functionality

**Components to build:**
- `SavedScreen.js`
- `EmptyState.js`

### 3.6 Reusable Components
Create these in `src/components/common/`:
- `Button.js` - Styled button with variants
- `Card.js` - Container with shadow/border
- `Badge.js` - Colored badge for tags
- `IconButton.js` - Icon with touchable
- `LoadingSpinner.js` - Activity indicator
- `ErrorMessage.js` - Error display component

---

## Phase 4: Navigation & State Management (Week 5-6)

### 4.1 Set Up Navigation
Create `src/navigation/AppNavigator.js`:
```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Stack = createNativeStackNavigator();
const Tab = createBottomTabNavigator();

function HomeTabs() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="List" component={HomeScreen} />
      <Tab.Screen name="Map" component={MapScreen} />
      <Tab.Screen name="Saved" component={SavedScreen} />
    </Tab.Navigator>
  );
}

export default function AppNavigator() {
  const [isOnboarded, setIsOnboarded] = useState(false);
  
  // Check onboarding status on mount
  
  return (
    <NavigationContainer>
      <Stack.Navigator>
        {!isOnboarded ? (
          <Stack.Screen 
            name="Onboarding" 
            component={OnboardingScreen}
            options={{ headerShown: false }}
          />
        ) : (
          <>
            <Stack.Screen 
              name="Home" 
              component={HomeTabs}
              options={{ headerShown: false }}
            />
            <Stack.Screen 
              name="ListingDetail" 
              component={ListingDetailScreen}
            />
          </>
        )}
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

### 4.2 Custom Hooks
Create `src/hooks/useListings.js`:
```javascript
import { useState, useEffect } from 'react';
import listingsData from '../data/listings.json';
import { checkEligibility } from '../utils/eligibility';
import { getUserProfile } from '../utils/storage';

export const useListings = () => {
  const [listings, setListings] = useState([]);
  const [filteredListings, setFilteredListings] = useState([]);
  const [userProfile, setUserProfile] = useState(null);
  const [filters, setFilters] = useState({
    bedrooms: null,
    maxRent: null,
    programs: [],
    neighborhoods: [],
  });

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    const profile = await getUserProfile();
    setUserProfile(profile);
    
    // Add eligibility flag to each listing
    const enrichedListings = listingsData.map(listing => ({
      ...listing,
      eligible: profile.income 
        ? checkEligibility(profile.income, profile.householdSize, listing.amiPercentage)
        : null,
    }));
    
    setListings(enrichedListings);
    setFilteredListings(enrichedListings);
  };

  const applyFilters = (newFilters) => {
    setFilters(newFilters);
    
    let filtered = [...listings];
    
    if (newFilters.bedrooms) {
      filtered = filtered.filter(l => l.bedrooms === newFilters.bedrooms);
    }
    
    if (newFilters.maxRent) {
      filtered = filtered.filter(l => l.rent <= newFilters.maxRent);
    }
    
    // Add more filter logic...
    
    setFilteredListings(filtered);
  };

  return {
    listings: filteredListings,
    userProfile,
    applyFilters,
    refreshListings: loadData,
  };
};
```

---

## Phase 5: Advanced Features (Week 6-8)

### 5.1 Transit Integration
- Use Expo Location to get user's location
- Calculate distance to nearest transit stops
- Display walking time estimates
- Option: Integrate with King County Metro API for real-time data

### 5.2 Search Functionality
- Text search across property names, addresses, neighborhoods
- Debounced input for performance
- Search history (optional)

### 5.3 Filter UI
- Bottom sheet or modal for advanced filters
- Multi-select for programs and neighborhoods
- Range sliders for rent and bedrooms
- "Clear all filters" button

### 5.4 Image Handling
- Implement image carousel with pagination dots
- Loading states for images
- Fallback images for missing photos
- Image caching for performance

### 5.5 Deep Linking
- Configure deep links for sharing specific listings
- Handle navigation from push notifications (future)

---

## Phase 6: Polish & Optimization (Week 8-9)

### 6.1 Performance Optimization
- Memoize expensive calculations
- Use FlatList for listing scrolling (virtualization)
- Optimize map marker rendering
- Image lazy loading
- Reduce bundle size

### 6.2 Error Handling
- Network error states
- Empty states for all screens
- Loading indicators
- Retry mechanisms
- Graceful degradation

### 6.3 Accessibility
- Screen reader support (accessibility labels)
- Sufficient color contrast
- Touch target sizes (minimum 44x44)
- Keyboard navigation support
- Dynamic text sizing support

### 6.4 UI Polish
- Smooth animations and transitions
- Consistent spacing and typography
- Loading skeletons
- Haptic feedback on interactions
- Pull-to-refresh animations

### 6.5 Testing
- Manual testing on iOS and Android
- Test on various screen sizes
- Test with different income levels
- Test edge cases (no listings, no internet, etc.)

---

## Phase 7: App Store Preparation (Week 9-10)

### 7.1 Legal Requirements
Create required documents:
- **Privacy Policy**: Explain data collection (location, user income)
- **Terms of Service**: Usage terms, disclaimers
- **Support Page**: Contact information, FAQ
- **Disclaimer**: "Information may not be current. Contact property directly to verify."

Host these at a simple website or GitHub Pages.

### 7.2 App Assets
Create required graphics:
- App icon (1024x1024 for iOS, various sizes for Android)
- Splash screen
- App Store screenshots (6.5", 5.5" iPhone and 12.9" iPad)
- Google Play screenshots (phone and tablet)
- Feature graphic (1024x500 for Google Play)
- Promotional images

### 7.3 App Store Listing
Write compelling copy:
- **Title**: "Evergreen Home - Affordable Housing Finder"
- **Subtitle** (iOS): "Find Seattle affordable housing you qualify for"
- **Description**: Highlight key features and benefits
- **Keywords** (iOS): affordable housing, Seattle housing, MFTE, income restricted
- **Category**: Lifestyle or Finance
- **Age Rating**: 4+ (no objectionable content)

### 7.4 Build Configuration
Update `app.json` for production:
```json
{
  "expo": {
    "version": "1.0.0",
    "privacy": "public",
    "ios": {
      "buildNumber": "1"
    },
    "android": {
      "versionCode": 1
    }
  }
}
```

### 7.5 Build for Production
```bash
# Install EAS CLI
npm install -g eas-cli

# Login to Expo account
eas login

# Configure EAS
eas build:configure

# Build for iOS
eas build --platform ios

# Build for Android
eas build --platform android

# Or build for both
eas build --platform all
```

### 7.6 Submit to App Stores

**Apple App Store:**
1. Create App Store Connect account ($99/year)
2. Create new app in App Store Connect
3. Upload build via EAS or manually
4. Fill out app information
5. Submit for review
6. Wait 24-48 hours for approval

**Google Play Store:**
1. Create Google Play Console account ($25 one-time)
2. Create new application
3. Upload APK/AAB
4. Fill out store listing
5. Submit for review
6. Usually approved within hours

---

## Phase 8: Post-Launch (Ongoing)

### 8.1 Analytics Setup (Optional)
- Integrate Expo Analytics or Firebase Analytics
- Track key metrics:
  - Daily active users
  - Listing views
  - Apply button clicks
  - Saved listings count
  - Search queries

### 8.2 User Feedback
- Monitor app store reviews
- Set up support email
- Create feedback mechanism in-app
- Track crash reports (Expo provides this)

### 8.3 Data Updates
- Plan for regular listing data updates
- Consider backend API for real-time data (future)
- Update AMI figures annually (HUD releases in March/April)

### 8.4 Feature Backlog (Future Versions)
**v1.1:**
- Push notifications for new listings
- User accounts (optional, for cross-device sync)
- Application tracking
- Listing history

**v1.2:**
- In-app messaging with property managers
- Waitlist management
- Document checklist
- Community reviews (moderated)

**v2.0:**
- Expand to other Washington cities (Tacoma, Spokane, Bellevue)
- Backend API with real-time data scraping
- AI-powered recommendations
- Integration with housing authority systems

---

## Key Technical Decisions & Best Practices

### Data Strategy
**For MVP:** Use static JSON file with curated listings (10-50 high-quality entries)
**Rationale:** Faster development, full control over quality, no backend costs
**Future:** Build scraper/API to aggregate from multiple sources

### State Management
**For MVP:** React hooks (useState, useContext) + AsyncStorage
**Rationale:** Simple, built-in, sufficient for MVP scope
**Future:** Consider Redux or Zustand if state becomes complex

### Backend Strategy
**For MVP:** No backend required (static data)
**Future Options:**
- Firebase (easy, scalable, real-time)
- Supabase (open-source, PostgreSQL)
- Custom API (Node.js/Express or Python/FastAPI)

### Testing Strategy
**For MVP:** Manual testing on physical devices
**Future:** Jest for unit tests, Detox for E2E tests

---

## Critical Path & MVP Definition

### Must-Have for Launch:
✅ Onboarding with income/household size capture  
✅ List view with eligibility filtering  
✅ Listing detail page with contact info  
✅ Saved listings functionality  
✅ Basic search and filters  
✅ 20+ quality, verified listings  
✅ Privacy policy and terms  
✅ App store presence (iOS or Android, at least one)

### Nice-to-Have (Can Launch Without):
- Map view (add in v1.1)
- Transit integration (add in v1.1)
- Advanced filters (can be basic initially)
- Polished animations
- Both iOS and Android simultaneously

---

## Risk Mitigation

### Technical Risks:
- **Map performance issues**: Start with list view, add map later
- **Data freshness**: Clear disclaimer that info may be outdated
- **API rate limits**: Use static data initially

### Legal Risks:
- **Incorrect eligibility calculations**: Include prominent disclaimer
- **Outdated listings**: Encourage users to verify with property
- **Privacy concerns**: Minimal data collection, clear privacy policy

### Market Risks:
- **Low adoption**: Focus on quality over quantity of listings
- **Competition**: Differentiate with better UX and local focus
- **Data sourcing**: Start with manual curation, automate later

---

## Success Metrics

### Launch Targets (First 3 Months):
- 500+ downloads
- 50+ active weekly users
- 10+ application clicks per week
- 4+ star average rating
- <5% crash rate

### Long-term Goals (Year 1):
- 5,000+ downloads
- 500+ weekly active users
- Partnerships with 2+ housing nonprofits
- Featured in local news/housing resources
- Expand to 100+ listings

---

## Resources & Documentation

### Key Documentation:
- Expo Docs: https://docs.expo.dev
- React Native Docs: https://reactnative.dev
- React Navigation: https://reactnavigation.org
- NativeWind: https://www.nativewind.dev

### Data Sources:
- Seattle Office of Housing: https://www.seattle.gov/housing
- AptFinder.org: https://www.aptfinder.org
- HUD AMI Data: https://www.huduser.gov
- King County Metro API: https://kingcounty.gov/metro

### Design Resources:
- Apple Human Interface Guidelines
- Material Design Guidelines
- Expo App Icon Generator

---

## Development Commands Reference

```bash
# Start development server
npx expo start

# Run on iOS simulator
npx expo start --ios

# Run on Android emulator
npx expo start --android

# Clear cache
npx expo start --clear

# Run on physical device (scan QR code with Expo Go app)
npx expo start

# Build for production
eas build --platform ios
eas build --platform android

# Submit to stores
eas submit --platform ios
eas submit --platform android

# Update app (OTA updates for non-native changes)
eas update
```

---

## Final Notes

This roadmap is designed to be **iterative**. Don't try to build everything at once. Focus on:

1. **Week 1-2**: Setup + data structure
2. **Week 3-4**: Basic UI (onboarding + list view)
3. **Week 5-6**: Core features (filters + detail page)
4. **Week 7-8**: Polish + testing
5. **Week 9-10**: App store prep + launch

**Remember:** A well-executed MVP with 20 great listings beats a feature-bloated app with 1000 outdated listings. Ship early, iterate based on user feedback.

Good luck building Evergreen Home! 🏡🌲
