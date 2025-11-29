# Testing Guide - Evergreen Home

## Overview
This document outlines comprehensive testing procedures for Evergreen Home to ensure a high-quality, bug-free user experience before launch.

---

## Testing Strategy

### Testing Pyramid
1. **Manual Testing** (70% of effort for MVP)
2. **Functional Testing** (20% of effort)
3. **Automated Testing** (10% of effort - future enhancement)

### Testing Phases
1. **Development Testing** - Continuous during development
2. **Integration Testing** - When features are combined
3. **System Testing** - Complete app testing
4. **Acceptance Testing** - Final pre-launch validation

---

## Manual Testing Checklist

### Onboarding Flow
- [ ] App opens without crashes
- [ ] Welcome screen displays correctly
- [ ] Income input accepts numeric values only
- [ ] Income input formats with commas and dollar sign
- [ ] Income input validates minimum value (e.g., $1)
- [ ] Income input validates maximum value (e.g., $500,000)
- [ ] Household size selector shows all options (1-8+)
- [ ] Household size selector highlights selected option
- [ ] "Get Started" button is disabled until both fields filled
- [ ] "Get Started" button navigates to home screen
- [ ] User data persists after app restart
- [ ] Can go back and change income/household size (if feature added)

### Home Screen - List View
- [ ] Listings display correctly with all information
- [ ] Images load properly (show placeholder if slow network)
- [ ] Eligibility badges show correct status
- [ ] Green "You qualify" badge for eligible listings
- [ ] Gray badge for ineligible listings
- [ ] Rent displays with dollar sign and comma formatting
- [ ] Bedroom/bathroom counts display correctly
- [ ] Studio apartments show "Studio" instead of "0 bed"
- [ ] Neighborhood and program labels display
- [ ] Heart icon saves/unsaves listings
- [ ] Saved heart is filled red, unsaved is outline gray
- [ ] Tapping listing card navigates to detail screen
- [ ] Pull-to-refresh works and shows loading indicator
- [ ] Scrolling is smooth (no lag with 50+ listings)
- [ ] Empty state shows when no listings (test with filters)

### Search Functionality
- [ ] Search bar accepts text input
- [ ] Search filters by property name
- [ ] Search filters by address
- [ ] Search filters by neighborhood
- [ ] Search is case-insensitive
- [ ] Search updates results in real-time
- [ ] Clear (X) button clears search
- [ ] No results message shows when appropriate
- [ ] Search persists when navigating back

### Filter System
- [ ] Filter button opens filter modal/sheet
- [ ] Bedroom filter works (0, 1, 2, 3, 4+)
- [ ] Rent range filter works (slider or input)
- [ ] Program type filter works (checkboxes)
- [ ] Neighborhood filter works (multi-select)
- [ ] "Eligible only" toggle works
- [ ] "Available only" toggle works
- [ ] Filters apply when "Apply" button pressed
- [ ] Filter count badge shows number of active filters
- [ ] "Clear all" button removes all filters
- [ ] Results update immediately after applying filters
- [ ] Filter modal closes after applying
- [ ] Filters persist when navigating away and back

### Listing Detail Screen
- [ ] Navigates from list card successfully
- [ ] All images load correctly
- [ ] Image carousel swipes left/right
- [ ] Pagination dots show current image
- [ ] Save/unsave heart button works
- [ ] Property name displays
- [ ] Full address displays
- [ ] Rent with /mo suffix displays
- [ ] Bed/bath/sqft displays
- [ ] Program badge displays
- [ ] Eligibility card shows correct status
- [ ] Income comparison shows user vs. max income
- [ ] Description text is readable
- [ ] Amenities list displays
- [ ] Pet policy displays (if applicable)
- [ ] Utilities information displays
- [ ] Transit information shows (if available)
- [ ] Contact buttons all work:
  - [ ] "Apply Online" opens browser
  - [ ] "Call" opens phone dialer
  - [ ] "Email" opens email app
- [ ] Management company name displays
- [ ] Last updated date shows
- [ ] Back button returns to previous screen
- [ ] Saved status persists across navigation

### Map Screen (if implemented)
- [ ] Map loads successfully
- [ ] User location shows (with permission)
- [ ] All listing pins display on map
- [ ] Pin colors indicate eligibility (green/gray)
- [ ] Pins cluster when zoomed out
- [ ] Tapping pin shows listing preview
- [ ] Tapping preview navigates to detail
- [ ] Map centers on Seattle area by default
- [ ] Zoom in/out works smoothly
- [ ] Map style is readable
- [ ] "My Location" button centers on user
- [ ] Toggle to switch to list view works

### Saved Listings Screen
- [ ] Displays all saved listings
- [ ] Shows same information as home screen
- [ ] Empty state shows when no saved listings
- [ ] Unsaving removes from list immediately
- [ ] Tapping listing navigates to detail
- [ ] Saved status persists across app restarts
- [ ] Can unsave from detail screen and see reflected

### Navigation
- [ ] Bottom tab bar shows on all main screens
- [ ] Active tab is highlighted
- [ ] Tab icons are clear and recognizable
- [ ] Switching tabs is instant (no lag)
- [ ] Navigation stack works correctly
- [ ] Back button always goes to previous screen
- [ ] Deep navigation doesn't break (e.g., home → detail → back)
- [ ] App state preserved when backgrounded

---

## Edge Cases & Error Handling

### Data Edge Cases
- [ ] No listings available (empty database)
- [ ] Single listing only
- [ ] 100+ listings (performance test)
- [ ] Listing with missing images
- [ ] Listing with single image
- [ ] Listing with 10+ images
- [ ] Very long property names (50+ chars)
- [ ] Very long addresses
- [ ] Very high rent ($5000+)
- [ ] Very low rent ($100)
- [ ] Zero bedrooms (studio)
- [ ] 5+ bedrooms
- [ ] Decimal bathrooms (1.5, 2.5)
- [ ] Null/missing square footage
- [ ] Special characters in descriptions
- [ ] Income limits not defined for all household sizes

### User Input Edge Cases
- [ ] Income: $0 (should show validation error)
- [ ] Income: $1 (valid but edge case)
- [ ] Income: $500,000+ (valid high income)
- [ ] Income: Non-numeric characters (should be blocked)
- [ ] Income: Negative numbers (should be blocked)
- [ ] Income: Copy-paste formatted number ($50,000)
- [ ] Household size: 1 person
- [ ] Household size: 8+ people
- [ ] Switching household size multiple times
- [ ] Changing income after saving listings

### Network & Performance
- [ ] App works offline (saved listings, stored data)
- [ ] App handles slow network gracefully
- [ ] Loading indicators show during data fetch
- [ ] Retry mechanism works after failed load
- [ ] App doesn't crash on network timeout
- [ ] Images load progressively
- [ ] Large images don't cause memory issues
- [ ] Scrolling remains smooth with images loading

### Permissions
- [ ] Location permission prompt shows (if using GPS)
- [ ] App works if location denied
- [ ] App explains why location is requested
- [ ] Can change permission in settings and app updates

### Device-Specific
- [ ] Works on iPhone SE (small screen)
- [ ] Works on iPhone 14 Pro Max (large screen)
- [ ] Works on iPad (tablet layout if implemented)
- [ ] Works on Android phones (various sizes)
- [ ] Notch/island doesn't obscure content
- [ ] Status bar is readable (light/dark)
- [ ] Works in light mode
- [ ] Works in dark mode (if implemented)
- [ ] Keyboard doesn't obscure input fields
- [ ] Safe area insets respected (no content under notch/home bar)

---

## Functional Testing Scenarios

### User Journey 1: First-Time User
1. Open app for first time
2. See onboarding screen
3. Enter income: $65,000
4. Select household size: 1 person
5. Tap "Get Started"
6. See home screen with filtered listings
7. Browse 3-5 listings
8. Save 2 listings
9. Open one listing detail
10. Tap "Apply Now"
11. Return to home
12. Open saved listings
13. Close app
14. Reopen app (should skip onboarding)

**Expected Results:**
- Smooth flow, no crashes
- Only eligible listings shown
- Saved listings persist
- External links open correctly

### User Journey 2: Job Seeker with Changing Income
1. Open app (already onboarded at $50k)
2. Browse listings eligible at 50% AMI
3. Get new job, income increases to $75k
4. Go to settings/profile
5. Update income to $75,000
6. See updated eligibility on all listings
7. Previously eligible listings now show as ineligible
8. New 80% AMI listings now show as eligible

**Expected Results:**
- Can update income easily
- Eligibility recalculates correctly
- Saved listings update eligibility status

### User Journey 3: Power User with Filters
1. Open app
2. Apply filters:
   - 1 bedroom only
   - Max rent $1,500
   - MFTE program only
   - Greenwood neighborhood
3. See 5 filtered results
4. Save 3 listings
5. Clear filters
6. See all listings again
7. Re-apply same filters
8. Results match previous

**Expected Results:**
- Filters work correctly in combination
- Results are consistent
- Clear filters restores full list

### User Journey 4: Property Application
1. Open app
2. Find desired listing
3. Open listing detail
4. Read full description
5. Verify eligibility
6. View all photos
7. Check transit info
8. Tap "Call" button → makes call
9. Return to app
10. Tap "Email" button → opens email
11. Return to app
12. Tap "Apply Online" → opens website
13. Save listing for later reference

**Expected Results:**
- All contact methods work
- App resumes correctly after external apps
- No data loss when switching apps

---

## Accessibility Testing

### Screen Reader (iOS: VoiceOver, Android: TalkBack)
- [ ] All buttons have labels
- [ ] All images have alt text
- [ ] Navigation is logical (tab order)
- [ ] Form fields are labeled
- [ ] Error messages are announced
- [ ] Can complete full user journey with screen reader

### Visual Accessibility
- [ ] Text meets minimum size (16px body text)
- [ ] Sufficient color contrast (4.5:1 for body text)
- [ ] Links/buttons distinguishable without color alone
- [ ] Touch targets minimum 44x44 points
- [ ] Important info not conveyed by color only

### Motor Accessibility
- [ ] All actions can be performed with one hand
- [ ] No double-tap required
- [ ] No precise timing required
- [ ] Swipe gestures have alternatives

---

## Performance Testing

### Metrics to Measure
```javascript
// Add performance monitoring
console.time('listingLoad');
// ... load listings
console.timeEnd('listingLoad');
// Target: < 500ms
```

### Performance Benchmarks
- [ ] App launch: < 3 seconds (cold start)
- [ ] App launch: < 1 second (warm start)
- [ ] Listing load: < 500ms
- [ ] Image load: < 2 seconds per image
- [ ] Search response: < 300ms
- [ ] Filter application: < 200ms
- [ ] Navigation transition: < 300ms
- [ ] Smooth scrolling: 60 FPS

### Memory Testing
- [ ] No memory leaks (use Xcode Instruments/Android Profiler)
- [ ] Memory usage: < 150MB during normal use
- [ ] App doesn't crash after extended use (30+ min)
- [ ] Background memory usage: < 50MB

### Battery Testing
- [ ] No excessive battery drain
- [ ] No background activity when app closed
- [ ] Location services stop when not in use

---

## Security Testing

### Data Security
- [ ] User income not exposed in logs
- [ ] No sensitive data in crash reports
- [ ] AsyncStorage data encrypted (if sensitive)
- [ ] No hardcoded API keys in code
- [ ] HTTPS for all network requests

### Privacy
- [ ] User can delete their data
- [ ] No data collection without consent
- [ ] Privacy policy easily accessible
- [ ] Location tracking only with permission
- [ ] No tracking across other apps

---

## Regression Testing

**When to Run:**
- After every major feature addition
- Before each release
- After bug fixes

**Core Flows to Always Test:**
1. Onboarding → Home → Detail → Apply
2. Search and filter
3. Save and unsave listings
4. Change user profile
5. Navigate between all screens

---

## Test Data Requirements

### Diverse Listings Dataset
Create test data with:
- Programs: MFTE (5), MHA (3), Section 8 (2), SHA (2), LIHTC (2), Other (1)
- AMI levels: 30% (2), 50% (3), 60% (4), 80% (5), 120% (1)
- Bedrooms: Studio (2), 1BR (6), 2BR (4), 3BR (2), 4BR (1)
- Rent range: $600 - $2,500
- Neighborhoods: Spread across 10+ neighborhoods
- Images: Some with 1 image, some with 5+
- Availability: 80% available, 20% unavailable

### Test User Profiles
```javascript
const testUsers = [
  { income: 28000, size: 1 },  // Very low income, qualifies for 30% AMI
  { income: 47000, size: 2 },  // Low income, qualifies for 50% AMI
  { income: 75000, size: 1 },  // Moderate, qualifies for 80% AMI
  { income: 90000, size: 4 },  // Family, qualifies for 60% AMI
  { income: 120000, size: 2 }, // Higher income, qualifies for 100%+ AMI only
];
```

---

## Bug Reporting Template

When finding bugs, document:
```markdown
## Bug Report

**Title:** [Brief description]

**Severity:** Critical / High / Medium / Low

**Device:** iPhone 14 Pro / Pixel 7 / etc.
**OS Version:** iOS 17.1 / Android 14 / etc.
**App Version:** 1.0.0

**Steps to Reproduce:**
1. Open app
2. Navigate to...
3. Tap on...
4. Observe...

**Expected Result:**
What should happen

**Actual Result:**
What actually happens

**Screenshots/Video:**
[Attach if applicable]

**Additional Notes:**
Any other relevant information
```

---

## Testing Tools

### Manual Testing
- Physical devices (iOS and Android)
- iOS Simulator (Xcode)
- Android Emulator (Android Studio)
- Expo Go app for quick testing

### Debugging Tools
```javascript
// Console logging
console.log('Listing data:', listing);

// React Native Debugger
// Install: https://github.com/jhen0409/react-native-debugger

// Expo Developer Tools
// Automatic when running: npx expo start
```

### Performance Monitoring
```javascript
// Expo performance monitoring (built-in)
// View in Expo dashboard

// React DevTools
// Install: npm install -g react-devtools
```

### Network Testing
- Charles Proxy (monitor network requests)
- Slow network simulation (iOS Simulator, Android Emulator)
- Airplane mode testing

---

## Test Automation (Future Enhancement)

### Unit Tests with Jest
```javascript
// Example: Test eligibility calculation
import { checkEligibility } from '../utils/eligibility';

test('User with $50k income qualifies for 60% AMI with household of 1', () => {
  expect(checkEligibility(50000, 1, 60)).toBe(true);
});

test('User with $100k income does not qualify for 60% AMI', () => {
  expect(checkEligibility(100000, 1, 60)).toBe(false);
});
```

### E2E Tests with Detox (Future)
```javascript
// Example: Test onboarding flow
describe('Onboarding', () => {
  it('should complete onboarding successfully', async () => {
    await element(by.id('incomeInput')).typeText('65000');
    await element(by.id('householdSize2')).tap();
    await element(by.id('getStartedButton')).tap();
    await expect(element(by.id('homeScreen'))).toBeVisible();
  });
});
```

---

## Pre-Launch Testing Checklist

**One Week Before Launch:**
- [ ] Complete all manual testing scenarios
- [ ] Test on minimum 3 iOS devices (different sizes)
- [ ] Test on minimum 3 Android devices (different manufacturers)
- [ ] Complete accessibility testing
- [ ] Verify all external links work
- [ ] Test with 50+ real listings
- [ ] Performance testing passed
- [ ] No critical or high-severity bugs

**Three Days Before Launch:**
- [ ] Final regression testing
- [ ] Test on app store builds (not dev builds)
- [ ] Verify analytics tracking works
- [ ] Test crash reporting
- [ ] Review app store screenshots match actual app
- [ ] Have 5 people test who haven't seen app before

**Launch Day:**
- [ ] Install from App Store/Play Store
- [ ] Complete one full user journey
- [ ] Monitor crash reports
- [ ] Test on personal device

---

## Post-Launch Monitoring

### First 24 Hours:
- Check crash rate (target: < 1%)
- Monitor app store reviews
- Test reported issues immediately
- Verify analytics data coming through

### First Week:
- Daily crash report review
- Track user completion rates for key flows
- Identify most common error messages
- Monitor performance metrics

### First Month:
- Weekly testing on new OS versions
- Test competitor features
- Gather user feedback
- Plan fixes based on data

---

## Testing Best Practices

1. **Test Early, Test Often:** Don't wait until the end
2. **Test Like a User:** Follow real user journeys
3. **Break Things:** Try to make the app crash
4. **Document Everything:** Record bugs systematically
5. **Test on Real Devices:** Simulators don't catch everything
6. **Test Edge Cases:** Where bugs usually hide
7. **Get Fresh Eyes:** Have others test who don't know the app
8. **Test Incrementally:** Test each feature as it's built
9. **Prioritize Critical Flows:** Onboarding and listing detail are most important
10. **Never Skip Regression:** Always test core flows before release

---

## Definition of Done

A feature is only "done" when:
- [ ] Functionality works as specified
- [ ] Tested on iOS and Android
- [ ] Tested on multiple screen sizes
- [ ] No console errors or warnings
- [ ] Accessibility checked
- [ ] Performance acceptable
- [ ] Edge cases handled
- [ ] Error states implemented
- [ ] Code reviewed
- [ ] Documented

---

## Emergency Testing (If Critical Bug Found)

**If a critical bug is reported after launch:**

1. **Reproduce:** Verify on your device
2. **Assess:** Determine severity and impact
3. **Fix:** Implement fix
4. **Test Fix:** Verify fix works
5. **Regression Test:** Ensure fix didn't break anything else
6. **Deploy:** Push OTA update or submit new build
7. **Monitor:** Watch for recurrence
8. **Document:** Record for future prevention

**Critical bugs require:**
- Same-day fix
- Full regression testing
- Multiple device testing
- Immediate deployment

Remember: Quality is more important than speed. A delayed launch with a polished app is better than a rushed launch with bugs. Test thoroughly! 🧪
