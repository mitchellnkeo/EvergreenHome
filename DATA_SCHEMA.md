# Data Schema Documentation - Evergreen Home

## Overview
This document defines all data structures, types, and constants used throughout the Evergreen Home application. Use this as the single source of truth for data modeling.

---

## Core Data Types

### Listing
Complete affordable housing listing object.

```typescript
interface Listing {
  // Identification
  id: string;                          // Unique identifier (UUID or incremental)
  name: string;                        // Property name
  
  // Location
  address: string;                     // Full street address
  city: string;                        // City (default: "Seattle")
  state: string;                       // State (default: "WA")
  zipCode: string;                     // ZIP code
  neighborhood: string;                // Neighborhood name
  lat: number;                         // Latitude (-90 to 90)
  lng: number;                         // Longitude (-180 to 180)
  
  // Property Details
  rent: number;                        // Monthly rent in USD
  bedrooms: number;                    // Number of bedrooms (0 = studio)
  bathrooms: number;                   // Number of bathrooms (can be decimal, e.g., 1.5)
  sqft: number | null;                 // Square footage (null if unknown)
  
  // Media
  images: string[];                    // Array of image URLs (first = primary)
  
  // Affordability
  program: AffordableHousingProgram;   // Program type
  amiPercentage: number;               // AMI percentage (30, 50, 60, 80, 120)
  incomeLimit: {                       // Income limits by household size
    [householdSize: number]: number;   // Max income for this household size
  };
  
  // Contact
  contactPhone: string;                // Phone number (format: "(206) 555-0123")
  contactEmail: string;                // Email address
  websiteUrl: string | null;           // Application/property website
  managementCompany: string;           // Property management company name
  
  // Status
  available: boolean;                  // Current availability
  availableDate: string | null;        // ISO date string when available
  
  // Additional Info
  amenities: string[];                 // List of amenities
  description: string;                 // Full property description
  petPolicy: PetPolicy;                // Pet policy details
  utilities: UtilityInfo;              // Utility inclusion info
  
  // Transit (optional, can be calculated)
  nearestTransit?: TransitStop[];      // Nearby transit stops
  
  // Metadata
  lastUpdated: string;                 // ISO date string of last update
  source: string;                      // Data source (e.g., "Seattle Office of Housing")
}
```

### AffordableHousingProgram
Program types available in Seattle/Washington.

```typescript
type AffordableHousingProgram = 
  | 'MFTE'      // Multifamily Tax Exemption
  | 'MHA'       // Mandatory Housing Affordability
  | 'Section8'  // Section 8 Housing Choice Voucher accepted
  | 'SHA'       // Seattle Housing Authority
  | 'LIHTC'     // Low-Income Housing Tax Credit
  | 'PSH'       // Permanent Supportive Housing
  | 'Other';    // Other affordable housing program
```

### PetPolicy
Pet allowance details.

```typescript
interface PetPolicy {
  allowed: boolean;                    // Whether pets are allowed
  types: ('cats' | 'dogs' | 'small')[];// Types of pets allowed
  deposit: number | null;              // Pet deposit amount
  monthlyFee: number | null;           // Monthly pet rent
  restrictions: string | null;         // Size/breed restrictions
}
```

### UtilityInfo
Utility inclusion information.

```typescript
interface UtilityInfo {
  included: string[];                  // e.g., ["water", "sewer", "garbage"]
  excluded: string[];                  // e.g., ["electricity", "gas", "internet"]
  notes: string | null;                // Additional utility notes
}
```

### TransitStop
Nearby transit information.

```typescript
interface TransitStop {
  name: string;                        // Stop name
  type: 'bus' | 'light-rail' | 'streetcar';
  routes: string[];                    // Route numbers/names
  distanceMiles: number;               // Distance in miles
  walkingMinutes: number;              // Estimated walking time
}
```

---

## User Profile

### UserProfile
Stored user information for eligibility calculation.

```typescript
interface UserProfile {
  income: number;                      // Annual household income in USD
  householdSize: number;               // Number of people (1-8+)
  amiPercentage?: number;              // Calculated AMI percentage
  onboardingComplete: boolean;         // Whether onboarding is done
  preferences?: UserPreferences;       // Optional user preferences
}
```

### UserPreferences
Optional user preferences (for future features).

```typescript
interface UserPreferences {
  savedSearches?: SavedSearch[];
  notificationsEnabled?: boolean;
  preferredNeighborhoods?: string[];
  maxCommute?: number;                 // Minutes
}
```

### SavedSearch
Saved search filter combination.

```typescript
interface SavedSearch {
  id: string;
  name: string;
  filters: ListingFilters;
  createdAt: string;                   // ISO date string
}
```

---

## Filtering & Search

### ListingFilters
All available filter options.

```typescript
interface ListingFilters {
  // Basic filters
  minBedrooms?: number;
  maxBedrooms?: number;
  minRent?: number;
  maxRent?: number;
  
  // Program filters
  programs?: AffordableHousingProgram[];
  
  // Location filters
  neighborhoods?: string[];
  maxDistance?: number;                // Miles from user location
  
  // Amenity filters
  requiredAmenities?: string[];
  
  // Availability
  availableOnly?: boolean;
  
  // Pet policy
  petsRequired?: boolean;
  
  // Eligibility
  eligibleOnly?: boolean;              // Show only user-eligible listings
}
```

### SearchQuery
Search query structure.

```typescript
interface SearchQuery {
  text: string;                        // Search text
  filters: ListingFilters;             // Applied filters
  sortBy: SortOption;                  // Sorting preference
  sortOrder: 'asc' | 'desc';
}

type SortOption = 
  | 'rent'
  | 'bedrooms'
  | 'distance'
  | 'newest'
  | 'relevance';
```

---

## AMI (Area Median Income) Data

### AMI Constants
2025 Seattle Area Median Income by household size.

```typescript
const AMI_2025_SEATTLE = {
  1: 93800,
  2: 107200,
  3: 120600,
  4: 133900,
  5: 144700,
  6: 155500,
  7: 166200,
  8: 177000,
} as const;

// For households larger than 8, add 8% per additional person
const AMI_INCREMENT_PERCENT = 0.08;
```

### AMI Percentage Tiers
Standard AMI percentage tiers used in Seattle affordable housing.

```typescript
const AMI_TIERS = [30, 50, 60, 65, 70, 80, 85, 100, 120] as const;

type AMITier = typeof AMI_TIERS[number];
```

---

## Program Information

### ProgramInfo
Detailed program information for user education.

```typescript
interface ProgramInfo {
  code: AffordableHousingProgram;
  name: string;
  description: string;
  eligibilityNotes: string;
  moreInfoUrl: string;
}

const PROGRAM_INFO: Record<AffordableHousingProgram, ProgramInfo> = {
  MFTE: {
    code: 'MFTE',
    name: 'Multifamily Tax Exemption',
    description: 'Seattle program requiring 20-25% of units in new buildings to be affordable in exchange for property tax exemption.',
    eligibilityNotes: 'Typically serves households earning 60-85% AMI. Income and household size restrictions apply.',
    moreInfoUrl: 'https://www.seattle.gov/housing/property-owners/multifamily-tax-exemption',
  },
  MHA: {
    code: 'MHA',
    name: 'Mandatory Housing Affordability',
    description: 'Seattle requirement that new development include affordable housing or pay a fee.',
    eligibilityNotes: 'Units typically at 60% AMI or below. Income limits vary by household size.',
    moreInfoUrl: 'https://www.seattle.gov/housing/property-owners/mandatory-housing-affordability',
  },
  Section8: {
    code: 'Section8',
    name: 'Section 8 Accepted',
    description: 'Property accepts Housing Choice Vouchers (Section 8) from eligible tenants.',
    eligibilityNotes: 'Must qualify for Section 8 voucher through King County Housing Authority or Seattle Housing Authority.',
    moreInfoUrl: 'https://www.kcha.org/housing/vouchers',
  },
  SHA: {
    code: 'SHA',
    name: 'Seattle Housing Authority',
    description: 'Property owned/managed by Seattle Housing Authority.',
    eligibilityNotes: 'Income limits typically at 30-80% AMI. Waitlists may apply.',
    moreInfoUrl: 'https://www.seattlehousing.org',
  },
  LIHTC: {
    code: 'LIHTC',
    name: 'Low-Income Housing Tax Credit',
    description: 'Federally funded program providing tax credits to developers who create affordable housing.',
    eligibilityNotes: 'Typically serves households at 50-60% AMI. Strict income verification required.',
    moreInfoUrl: 'https://www.commerce.wa.gov/building-infrastructure/housing/housing-trust-fund/',
  },
  PSH: {
    code: 'PSH',
    name: 'Permanent Supportive Housing',
    description: 'Affordable housing with supportive services for individuals experiencing homelessness or with disabilities.',
    eligibilityNotes: 'Eligibility through referral only. Contact property for details.',
    moreInfoUrl: 'https://www.seattlehousing.org/housing/permanent-supportive-housing',
  },
  Other: {
    code: 'Other',
    name: 'Other Affordable Housing',
    description: 'Affordable housing through other programs or private initiatives.',
    eligibilityNotes: 'Eligibility varies. Contact property directly.',
    moreInfoUrl: '',
  },
};
```

---

## Neighborhoods

### Seattle Neighborhood List
Standardized neighborhood names for filtering.

```typescript
const SEATTLE_NEIGHBORHOODS = [
  'Ballard',
  'Belltown',
  'Capitol Hill',
  'Central District',
  'Columbia City',
  'Delridge',
  'Downtown',
  'Eastlake',
  'Fremont',
  'Georgetown',
  'Green Lake',
  'Greenwood',
  'International District',
  'Lake City',
  'Laurelhurst',
  'Madison Park',
  'Magnolia',
  'Northgate',
  'Phinney Ridge',
  'Pioneer Square',
  'Queen Anne',
  'Rainier Beach',
  'Rainier Valley',
  'Ravenna',
  'Roosevelt',
  'South Lake Union',
  'University District',
  'Wallingford',
  'West Seattle',
] as const;

type SeattleNeighborhood = typeof SEATTLE_NEIGHBORHOODS[number];
```

---

## Amenities

### Standard Amenities List
Standardized amenity names for consistency.

```typescript
const STANDARD_AMENITIES = [
  // Building
  'Elevator',
  'Secured Entry',
  'On-Site Laundry',
  'In-Unit Laundry',
  'Storage',
  'Bike Storage',
  'Parking',
  'Garage Parking',
  
  // Unit
  'Dishwasher',
  'Air Conditioning',
  'Balcony/Patio',
  'Walk-In Closet',
  'Hardwood Floors',
  'Carpet',
  
  // Community
  'Fitness Center',
  'Community Room',
  'Rooftop Deck',
  'Courtyard',
  'BBQ Area',
  'Playground',
  
  // Other
  'Pet Friendly',
  'Wheelchair Accessible',
  'Near Transit',
  'Near Parks',
] as const;

type Amenity = typeof STANDARD_AMENITIES[number];
```

---

## Storage Keys

### AsyncStorage Keys
All keys used for local storage.

```typescript
const STORAGE_KEYS = {
  // User Profile
  USER_INCOME: '@evergreen_user_income',
  HOUSEHOLD_SIZE: '@evergreen_household_size',
  ONBOARDING_COMPLETE: '@evergreen_onboarding',
  USER_PREFERENCES: '@evergreen_preferences',
  
  // Saved Data
  SAVED_LISTINGS: '@evergreen_saved_listings',
  SAVED_SEARCHES: '@evergreen_saved_searches',
  SEARCH_HISTORY: '@evergreen_search_history',
  
  // App State
  LAST_VIEWED_LISTING: '@evergreen_last_viewed',
  LAST_SYNC: '@evergreen_last_sync',
  
  // Cache
  LISTINGS_CACHE: '@evergreen_listings_cache',
  CACHE_TIMESTAMP: '@evergreen_cache_timestamp',
} as const;
```

---

## API Response Types (Future Backend)

### ListingsResponse
API response for fetching listings.

```typescript
interface ListingsResponse {
  success: boolean;
  data: Listing[];
  total: number;
  page: number;
  pageSize: number;
  lastUpdated: string;                 // ISO date string
}
```

### ErrorResponse
Standard error response.

```typescript
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
  };
}
```

---

## Validation Rules

### Listing Validation
Rules for validating listing data.

```typescript
const VALIDATION_RULES = {
  listing: {
    id: {
      required: true,
      type: 'string',
      minLength: 1,
    },
    name: {
      required: true,
      type: 'string',
      minLength: 2,
      maxLength: 200,
    },
    rent: {
      required: true,
      type: 'number',
      min: 0,
      max: 10000,
    },
    bedrooms: {
      required: true,
      type: 'number',
      min: 0,
      max: 10,
    },
    bathrooms: {
      required: true,
      type: 'number',
      min: 0,
      max: 10,
    },
    lat: {
      required: true,
      type: 'number',
      min: -90,
      max: 90,
    },
    lng: {
      required: true,
      type: 'number',
      min: -180,
      max: 180,
    },
    amiPercentage: {
      required: true,
      type: 'number',
      enum: AMI_TIERS,
    },
  },
  userProfile: {
    income: {
      required: true,
      type: 'number',
      min: 0,
      max: 1000000,
    },
    householdSize: {
      required: true,
      type: 'number',
      min: 1,
      max: 15,
    },
  },
};
```

---

## Example Data

### Sample Listing
Complete example of a properly formatted listing.

```json
{
  "id": "mfte-greenwood-001",
  "name": "Greenwood Plaza Apartments",
  "address": "8515 Greenwood Ave N",
  "city": "Seattle",
  "state": "WA",
  "zipCode": "98103",
  "neighborhood": "Greenwood",
  "lat": 47.6911,
  "lng": -122.3542,
  "rent": 1200,
  "bedrooms": 1,
  "bathrooms": 1,
  "sqft": 650,
  "images": [
    "https://example.com/greenwood-exterior.jpg",
    "https://example.com/greenwood-kitchen.jpg",
    "https://example.com/greenwood-bedroom.jpg"
  ],
  "program": "MFTE",
  "amiPercentage": 80,
  "incomeLimit": {
    "1": 75040,
    "2": 85760,
    "3": 96480,
    "4": 107120,
    "5": 115760,
    "6": 124400,
    "7": 132960,
    "8": 141600
  },
  "contactPhone": "(206) 555-0123",
  "contactEmail": "leasing@greenwoodplaza.com",
  "websiteUrl": "https://greenwoodplaza.com/apply",
  "managementCompany": "Seattle Property Management LLC",
  "available": true,
  "availableDate": "2025-12-01",
  "amenities": [
    "On-Site Laundry",
    "Secured Entry",
    "Bike Storage",
    "Near Transit",
    "Hardwood Floors"
  ],
  "description": "Beautiful one-bedroom apartment in the heart of Greenwood. Features hardwood floors, updated kitchen with dishwasher, and large windows with natural light. Walking distance to shops, restaurants, and transit. MFTE income-restricted unit.",
  "petPolicy": {
    "allowed": true,
    "types": ["cats", "small"],
    "deposit": 500,
    "monthlyFee": 35,
    "restrictions": "Dogs under 20 lbs only, max 2 pets"
  },
  "utilities": {
    "included": ["water", "sewer", "garbage"],
    "excluded": ["electricity", "gas", "internet"],
    "notes": "Average electricity cost is $40-60/month"
  },
  "nearestTransit": [
    {
      "name": "Greenwood Ave N & N 85th St",
      "type": "bus",
      "routes": ["5", "28"],
      "distanceMiles": 0.1,
      "walkingMinutes": 2
    }
  ],
  "lastUpdated": "2025-11-20T10:30:00Z",
  "source": "Seattle Office of Housing"
}
```

### Sample UserProfile
Example user profile data.

```json
{
  "income": 65000,
  "householdSize": 1,
  "amiPercentage": 69,
  "onboardingComplete": true,
  "preferences": {
    "notificationsEnabled": false,
    "preferredNeighborhoods": ["Greenwood", "Ballard", "Fremont"],
    "maxCommute": 30
  }
}
```

---

## Data Migration Strategy

### Version 1.0 to 1.1
If data schema changes in future versions:

```typescript
interface MigrationConfig {
  fromVersion: string;
  toVersion: string;
  migrations: Array<(data: any) => any>;
}

const MIGRATIONS: MigrationConfig[] = [
  {
    fromVersion: '1.0',
    toVersion: '1.1',
    migrations: [
      // Example: Add new field with default value
      (listing: any) => ({
        ...listing,
        petPolicy: listing.petPolicy || {
          allowed: false,
          types: [],
          deposit: null,
          monthlyFee: null,
          restrictions: null,
        },
      }),
    ],
  },
];
```

---

## Data Quality Checklist

Before adding a listing to the database, verify:

- [ ] All required fields are present
- [ ] Coordinates are valid (within Seattle area: ~47.4-47.8 lat, -122.2-122.5 lng)
- [ ] Rent is reasonable ($0-$5000 range)
- [ ] AMI percentage matches a standard tier (30, 50, 60, 80, 120)
- [ ] Income limits are calculated correctly
- [ ] Contact information is valid and working
- [ ] Images are accessible URLs
- [ ] Program type is appropriate
- [ ] Neighborhood name matches standardized list
- [ ] lastUpdated is recent (within 90 days)

---

## Testing Data

### Mock Listings for Development
Create varied test data covering:

1. **Edge cases**: Studio (0 bed), large unit (4+ bed), very low rent, high AMI
2. **All programs**: At least one listing per program type
3. **All neighborhoods**: Coverage across Seattle
4. **Availability**: Mix of available and unavailable
5. **Eligibility**: Mix that matches different income levels

### Sample Test User Profiles

```typescript
const TEST_USERS = {
  veryLowIncome: { income: 28000, householdSize: 1 },  // ~30% AMI
  lowIncome: { income: 47000, householdSize: 2 },      // ~44% AMI
  moderateIncome: { income: 75000, householdSize: 1 }, // ~80% AMI
  largeHousehold: { income: 90000, householdSize: 6 }, // ~58% AMI
};
```

---

## Constants Reference

All constants in one place for easy import:

```typescript
export const CONSTANTS = {
  AMI_2025_SEATTLE,
  AMI_TIERS,
  AMI_INCREMENT_PERCENT,
  PROGRAM_INFO,
  SEATTLE_NEIGHBORHOODS,
  STANDARD_AMENITIES,
  STORAGE_KEYS,
  VALIDATION_RULES,
} as const;
```

This data schema provides a complete foundation for building the Evergreen Home app with consistent, validated data structures.
