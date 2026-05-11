Skill-Exchange App - Complete Development Package
I'll create a comprehensive SOP document and detailed screen specifications for your Skill-Exchange Android app. This will be ready to hand over to your development team (Antigravity).

📋 PART 1: DETAILED SCREEN SPECIFICATIONS WITH PROMPTS
Screen 1: Splash Screen
Screen ID: SCR-001
Navigation: Auto-navigates to Login/Onboarding

UI Prompt:

text

Design a warm, welcoming splash screen for a rural artisan app with:
- App logo centered with traditional Indian craft elements
- Tagline: "Skills बदलो, समृद्धि पाओ" (Exchange Skills, Gain Prosperity)
- Warm color palette: terracotta (#E07856), saffron (#FF9933), leaf green (#4CAF50)
- Subtle animated pulse effect on logo
- Progress indicator at bottom
- Minimum display time: 2 seconds
Backend Requirements:

Check authentication status (Firebase Auth)
Preload skill taxonomy from Firestore
Initialize FCM token
Check network connectivity
Load cached user data from Room DB if offline
Connected To:

✅ Login Screen (SCR-002) if not authenticated
✅ Home Screen (SCR-005) if already logged in
Screen 2: Phone Authentication / Login
Screen ID: SCR-002
Navigation: From Splash → To OTP Verification

UI Prompt:

text

Create a low-literacy friendly phone authentication screen:
- Large phone icon at top
- Country code selector (+91 for India) with flag
- 10-digit phone number input with large, clear digits (24sp)
- Voice-over instruction button (icon: speaker)
- Primary CTA: "आगे बढ़ें" (Continue) button - full width, rounded corners
- Secondary text: "हम आपको OTP भेजेंगे" (We'll send you an OTP)
- Illustrative image showing a person with a phone
- Minimum touch target: 48dp
- Support for both English and Kannada
Backend Requirements:

Kotlin

// Firebase Phone Auth
FirebaseAuth.getInstance().apply {
    setLanguageCode("kn") // Kannada
    verifyPhoneNumber(
        phoneNumber = "+91${userInput}",
        timeout = 60L,
        unit = TimeUnit.SECONDS,
        callbacks = PhoneAuthCallbacks
    )
}
Firestore Security Rules:

JavaScript

match /users/{userId} {
  allow create: if request.auth != null;
  allow read, update: if request.auth.uid == userId;
}
Connected To:

✅ OTP Verification (SCR-003)
❌ Back: Exit app confirmation
Screen 3: OTP Verification
Screen ID: SCR-003
Navigation: From Login → To Profile Setup or Home

UI Prompt:

text

Design an OTP verification screen with:
- Header text: "OTP डालें" (Enter OTP)
- Subheader: Phone number displayed with edit option
- 6 boxes for OTP digits (single digit per box)
- Auto-advance focus between boxes
- Countdown timer: "Resend in 00:59"
- Resend OTP button (disabled until timer expires)
- Voice call fallback option
- Verify button (enabled only when 6 digits entered)
- Error message space for invalid OTP
- Loading state during verification
Backend Requirements:

Kotlin

// OTP Verification
credential = PhoneAuthProvider.getCredential(verificationId, otpCode)
FirebaseAuth.getInstance().signInWithCredential(credential)
    .addOnCompleteListener { task ->
        if (task.isSuccessful) {
            val user = task.result?.user
            checkUserProfileExists(user.uid)
        }
    }
Connected To:

✅ Profile Setup (SCR-004) if new user
✅ Home Screen (SCR-005) if returning user
⬅️ Back to Login to edit phone number
Screen 4: Profile Setup / Onboarding
Screen ID: SCR-004
Navigation: From OTP (new users) → To Home

UI Prompt:

text

Create a multi-step profile creation flow with progress indicator:

STEP 1 - Basic Info:
- Profile photo upload (optional) with AI avatar generation option
- Name input field with voice-to-text icon
- Village/Taluka dropdown with autocomplete
- "Next" button

STEP 2 - Skills Selection:
- Grid of skill category cards with icons:
  • Electrical (⚡)
  • Plumbing (🔧)
  • Carpentry (🪚)
  • Masonry (🧱)
  • Welding (🔥)
  • Agricultural Mechanics (🚜)
  • General Labor (💪)
  • Tailoring (✂️)
- Multi-select capability (up to 5 skills)
- For each selected skill:
  * Proficiency level slider: Beginner/Intermediate/Expert
  * Years of experience input
- "Next" button

STEP 3 - Welcome:
- Congratulations message
- Trust Score badge (Bronze - starting at 0)
- Initial Skill Points: 0
- Tutorial video option
- "Start Exchanging" CTA

Design: Warm colors, large icons (48dp), high contrast text
Backend Requirements:

Kotlin

// Create User Profile in Firestore
val userProfile = hashMapOf(
    "userId" to firebaseAuth.currentUser.uid,
    "name" to name,
    "phoneNumber" to phoneNumber,
    "village" to selectedVillage,
    "taluka" to selectedTaluka,
    "skills" to selectedSkills.map { 
        mapOf(
            "category" to it.category,
            "proficiency" to it.level,
            "yearsExp" to it.years
        )
    },
    "trustScore" to 0,
    "skillPoints" to 0,
    "profilePhotoUrl" to photoUrl,
    "createdAt" to FieldValue.serverTimestamp(),
    "location" to GeoPoint(latitude, longitude)
)

db.collection("users")
    .document(userId)
    .set(userProfile)
Connected To:

✅ Home Screen (SCR-005)
Screen 5: Home Screen / Feed
Screen ID: SCR-005
Navigation: Main hub of the app

UI Prompt:

text

Design a dashboard-style home screen with:

TOP SECTION (Fixed):
- Profile avatar (left) - tappable → My Profile (SCR-011)
- Trust Score badge with numeric value (0-100) and visual tier
- Skill Points balance in prominent badge
- Notification bell icon (right)

QUICK ACTIONS BAR:
- Large FAB: "+ Post Need" button (primary action)
- "Browse Needs" button
- "My Swaps" button

RECOMMENDED FOR YOU (GenAI Section):
- Header: "आपके लिए" (For You)
- Horizontal scrollable cards showing matched Needs
- Each card shows:
  * Need title
  * Skill required (with icon)
  * Distance
  * Poster's trust score
  * Hours needed
  * "Make Offer" quick action

NEARBY NEEDS FEED:
- Vertical list of Need cards
- Filter chips at top: All Skills / Distance / Urgency
- Real-time badge: "2 new" indicator
- Pull-to-refresh

BOTTOM NAVIGATION:
- Home (active)
- Skill Board (SCR-006)
- My Swaps (SCR-010)
- Profile (SCR-011)

Design: Card-based layout, earthy colors, clear hierarchy
Backend Requirements:

Kotlin

// Real-time Firestore listener for Needs
db.collection("needs")
    .whereEqualTo("status", "open")
    .orderBy("timestamp", Query.Direction.DESCENDING)
    .limit(50)
    .addSnapshotListener { snapshot, error ->
        snapshot?.documents?.map { it.toNeed() }
    }

// GenAI Recommendations
suspend fun getRecommendedNeeds(userSkills: List<String>): List<Need> {
    val gemini = GeminiService()
    return gemini.matchNeedsToSkills(
        availableNeeds = allOpenNeeds,
        userSkills = userSkills
    )
}

// Offline caching
@Dao
interface NeedDao {
    @Query("SELECT * FROM needs WHERE status = 'open' ORDER BY timestamp DESC LIMIT 50")
    fun getCachedNeeds(): Flow<List<NeedEntity>>
}
Connected To:

✅ Post Need (SCR-007)
✅ Skill Board (SCR-006)
✅ Need Detail (SCR-008)
✅ My Profile (SCR-011)
✅ My Swaps (SCR-010)
Screen 6: Skill Board (Need Board)
Screen ID: SCR-006
Navigation: From Bottom Nav or Home

UI Prompt:

text

Create a filterable browse screen:

TOP BAR:
- Search icon (tap to expand search field)
- Filter button (shows active filter count)

FILTER CHIPS (Horizontal scroll):
- All Skills
- Electrical ⚡
- Plumbing 🔧
- Carpentry 🪚
- Masonry 🧱
- Welding 🔥
- (show only 4, "+3 more" expands)
- Distance: 5km / 10km / 25km / Any
- Urgency: Urgent 🔴 / Flexible 🟢

NEED CARDS (Vertical list):
Each card contains:
- Skill category icon (top-left)
- Need title (bold, 18sp)
- Poster name + Trust Score badge
- Distance with location pin icon
- Hours needed: "2 घंटे" with clock icon
- Description snippet (2 lines, truncated)
- Posted time: "2 hours ago"
- "Make Offer" button (right)

EMPTY STATE:
- Illustration of person looking
- "कोई आवश्यकता नहीं मिली" (No needs found)
- Suggestion to adjust filters

REAL-TIME INDICATOR:
- New posts badge on filter bar
- Subtle animation when new Need appears

Design: Scannable, high contrast, clear CTAs
Backend Requirements:

Kotlin

// Compound Firestore query with filters
db.collection("needs")
    .whereEqualTo("status", "open")
    .whereEqualTo("skillRequired", selectedSkill) // if filtered
    .orderBy("timestamp", Query.Direction.DESCENDING)
    .startAfter(lastVisible) // pagination
    .limit(20)
    .get()

// Distance filtering (client-side after fetch)
fun filterByDistance(needs: List<Need>, maxKm: Double): List<Need> {
    return needs.filter { need ->
        calculateDistance(
            userLocation,
            need.location
        ) <= maxKm
    }
}
Connected To:

✅ Need Detail (SCR-008)
✅ Filter Bottom Sheet (SCR-006A)
⬅️ Back to Home
Screen 7: Post Need (GenAI Composer)
Screen ID: SCR-007
Navigation: From Home FAB

UI Prompt:

text

Design a voice-first posting interface:

HEADER:
- "अपनी आवश्यकता बताएं" (Tell Your Need)
- Close button (×)

VOICE INPUT SECTION (Primary):
- Large microphone button (center)
- "बोलकर बताएं" (Speak to tell)
- Animated waveform during recording
- Transcribed text appears below in real-time
- Kannada/Hindi/English auto-detected

TEXT INPUT (Alternative):
- "या लिखें" (Or type)
- Multi-line text field
- Voice-to-text still available

AI ASSISTANCE:
- Auto-detected skill category (editable dropdown)
- Suggested hours estimate (editable)
- Location auto-filled (with edit option)

MANUAL FIELDS:
- Need title (auto-generated from voice, editable)
- Skill required (dropdown with icons)
- Estimated hours needed (number picker: 1-24)
- Urgency toggle: Urgent / Flexible
- Location: Current / Select on map
- Add photo (optional)

PREVIEW CARD:
- Shows how Need will appear to others

CTA BUTTONS:
- "Post Need" (primary, full width)
- "Save as Draft" (secondary)

Design: Large touch targets, clear voice affordance
Backend Requirements:

Kotlin

// GenAI Voice-to-Text
suspend fun transcribeVoice(audioBytes: ByteArray): String {
    return gemini.transcribeAudio(
        audio = audioBytes,
        language = "kn" // Kannada
    )
}

// GenAI Auto-tagging
suspend fun extractNeedDetails(text: String): NeedDetails {
    val prompt = """
    Extract from this text:
    - Skill category (Electrical/Plumbing/Carpentry/Masonry/Welding/Agricultural/Labor/Tailoring)
    - Estimated hours needed
    - Urgency level
    
    Text: $text
    """
    return gemini.generateContent(prompt).parseAsNeedDetails()
}

// Create Need in Firestore
val need = hashMapOf(
    "needId" to UUID.randomUUID().toString(),
    "posterId" to currentUserId,
    "title" to title,
    "description" to description,
    "skillRequired" to skillCategory,
    "hoursNeeded" to hours,
    "location" to GeoPoint(lat, lng),
    "urgency" to urgency,
    "status" to "open",
    "timestamp" to FieldValue.serverTimestamp()
)

db.collection("needs").add(need)

// Send FCM to matched users
sendNotificationToMatchedUsers(skillCategory, location)
Connected To:

✅ Home Screen (after posting)
✅ My Needs (SCR-011B)
❌ Discard draft confirmation dialog
Screen 8: Need Detail
Screen ID: SCR-008
Navigation: From Skill Board / Home Feed

UI Prompt:

text

Design a detailed Need view:

HEADER:
- Back button
- Share button
- Bookmark button

POSTER INFO SECTION:
- Profile photo
- Name
- Trust Score badge (prominent)
- "View Profile" link
- Distance from you

NEED DETAILS:
- Skill category icon + name
- Need title (large, bold)
- Full description (expandable if long)
- Hours needed: "⏱️ 3 घंटे"
- Urgency indicator: 🔴 Urgent / 🟢 Flexible
- Posted time
- Location: Village name + "View on map" link

CURRENT OFFERS (if any):
- Count: "5 offers received"
- Scroll preview of offers (for poster only)

ACTION BUTTONS:
If user is NOT the poster:
- Primary: "Make Swap Offer" (full width)
- Secondary: "Ask Question" (opens chat)

If user IS the poster:
- "View All Offers" button
- "Edit Need" / "Close Need" options menu

SIMILAR NEEDS (bottom):
- "Similar needs nearby" carousel

Design: Clear hierarchy, trust signals prominent
Backend Requirements:

Kotlin

// Fetch Need with poster details
db.collection("needs")
    .document(needId)
    .get()
    .addOnSuccessListener { needDoc ->
        val need = needDoc.toNeed()
        
        // Get poster profile
        db.collection("users")
            .document(need.posterId)
            .get()
    }

// Fetch offers count
db.collection("offers")
    .whereEqualTo("needId", needId)
    .get()
    .addOnSuccessListener { offers ->
        offerCount = offers.size()
    }
Connected To:

✅ Make Offer / Chat (SCR-009)
✅ Poster Profile (SCR-011)
✅ Map View (SCR-008A)
⬅️ Back to previous screen
Screen 9: Make Offer & Chat
Screen ID: SCR-009
Navigation: From Need Detail

UI Prompt:

text

Design a hybrid offer + chat interface:

TOP SECTION (Pinned Offer Summary):
- Need title (abbreviated)
- Your offer terms (editable until accepted):
  * Service you'll provide (dropdown)
  * Hours you'll give (number)
  * "Is this fair?" GenAI button
- Status badge: Pending / Accepted / Declined

CHAT THREAD:
- Messages grouped by date
- Sender avatar + name
- Message bubbles (sent: right, received: left)
- Timestamp
- Support for text + voice messages + images

GENAI ASSISTANT (Floating button):
- Icon: ✨ AI
- Tap to ask: "Is this swap fair?"
- AI response appears as system message

INPUT BAR (Bottom):
- Text input field
- Voice message button (hold to record)
- Send button (adaptive icon based on input)

QUICK ACTIONS (Above keyboard):
- "Propose different terms"
- "Accept their offer"
- "Decline politely"

NOTIFICATIONS:
- Push when new message received
- In-app badge when other party responds

Design: WhatsApp-inspired, familiar UX
Backend Requirements:

Kotlin

// Create Offer
val offer = hashMapOf(
    "offerId" to UUID.randomUUID().toString(),
    "needId" to needId,
    "offererId" to currentUserId,
    "serviceOffered" to selectedSkill,
    "hoursOffered" to hours,
    "status" to "pending",
    "chatThread" to arrayListOf<Message>(),
    "createdAt" to FieldValue.serverTimestamp()
)

db.collection("offers").add(offer)

// Real-time chat listener
db.collection("offers")
    .document(offerId)
    .addSnapshotListener { doc, _ ->
        val messages = doc?.get("chatThread") as List<Message>
        updateChatUI(messages)
    }

// GenAI Fair Swap Analysis
suspend fun analyzeFairness(
    serviceOffered: String,
    hoursOffered: Int,
    serviceNeeded: String,
    hoursNeeded: Int
): FairnessScore {
    val prompt = """
    Analyze swap fairness:
    Offered: $hoursOffered hours of $serviceOffered
    Needed: $hoursNeeded hours of $serviceNeeded
    
    Is this equitable? Consider skill complexity.
    """
    return gemini.generateContent(prompt).parseFairnessScore()
}

// FCM to Need poster
sendOfferNotification(needPosterId, offerDetails)
Connected To:

✅ Swap Tracker (SCR-010) after acceptance
✅ Need Detail (SCR-008)
⬅️ Back: Save draft message
Screen 10: My Swaps / Swap Tracker
Screen ID: SCR-010
Navigation: From Bottom Nav or Home

UI Prompt:

text

Design a swap management dashboard:

TABS:
- Active Swaps
- Pending Confirmation
- Completed
- History

ACTIVE SWAPS TAB:
Each swap card contains:
- Other party's name + avatar
- Trust score badge
- Services being exchanged:
  "You give: 2hrs Plumbing ⟷ You receive: 2hrs Electrical"
- Progress tracker:
  ☐ Your task completed
  ☐ Their task completed
- Days remaining: "2 days left to confirm"
- Action buttons:
  * "Mark My Work Complete"
  * "Chat with [Name]"
  * "View Details"

PENDING CONFIRMATION TAB:
- Swaps awaiting dual confirmation
- 72-hour countdown timer
- "Confirm Completion" CTA (large)
- "Raise Dispute" option (subtle)

COMPLETED TAB:
- Success badge
- Skill points awarded
- Trust score change: +1
- "Rate Experience" (if not rated)
- Transaction ID

HISTORY TAB:
- Chronological list
- Filter by: Month / Skill / Partner
- Export option (CSV for records)

EMPTY STATES:
- Friendly illustrations
- "Start your first swap!" CTA

Design: Status-driven colors, clear progress indicators
Backend Requirements:

Kotlin

// Fetch user's swaps
db.collection("swaps")
    .whereArrayContains("participants", currentUserId)
    .orderBy("createdAt", Query.Direction.DESCENDING)
    .addSnapshotListener { snapshot, _ ->
        val swaps = snapshot?.toSwaps()
        categorizeSwaps(swaps)
    }

// Mark work complete (single confirmation)
db.collection("swaps")
    .document(swapId)
    .update("confirmedBy${currentUserRole}", true)
    .addOnSuccessListener {
        checkBothConfirmed(swapId)
    }

// Process swap completion (both confirmed)
fun processSwapCompletion(swapId: String) {
    db.runTransaction { transaction ->
        val swapRef = db.collection("swaps").document(swapId)
        val swap = transaction.get(swapRef).toSwap()
        
        if (swap.confirmedByA && swap.confirmedByB) {
            // Update Trust Scores
            transaction.update(
                db.collection("users").document(swap.partyA),
                "trustScore", FieldValue.increment(1)
            )
            transaction.update(
                db.collection("users").document(swap.partyB),
                "trustScore", FieldValue.increment(1)
            )
            
            // Update Skill Points
            transaction.update(
                db.collection("users").document(swap.partyA),
                "skillPoints", FieldValue.increment(swap.hoursA.toLong())
            )
            transaction.update(
                db.collection("users").document(swap.partyB),
                "skillPoints", FieldValue.increment(swap.hoursB.toLong())
            )
            
            // Mark swap complete
            transaction.update(swapRef, mapOf(
                "status" to "completed",
                "completedAt" to FieldValue.serverTimestamp()
            ))
        }
    }
}

// Dispute handling (72-hour window)
suspend fun checkDisputeWindow(swapId: String) {
    val swap = getSwap(swapId)
    val hoursSinceCreation = swap.getHoursSinceCreation()
    
    if (hoursSinceCreation > 72 && 
        (!swap.confirmedByA || !swap.confirmedByB)) {
        
        swap.status = "disputed"
        notifyModerator(swapId)
        generateDisputeSummary(swapId)
    }
}
Connected To:

✅ Chat (SCR-009)
✅ Dispute Resolution (SCR-010A)
✅ Partner Profile (SCR-011)
✅ Swap Detail (SCR-010B)
Screen 11: My Profile
Screen ID: SCR-011
Navigation: From Bottom Nav / Profile Avatar

UI Prompt:

text

Design a comprehensive profile screen:

HEADER SECTION:
- Cover photo (gradient or pattern)
- Profile photo (editable)
- Name
- Phone number (masked: +91 ••••• 12345)
- Village, Taluka
- Member since date

TRUST & POINTS DASHBOARD:
- Large Trust Score badge:
  * Numeric (0-100)
  * Visual tier: Bronze/Silver/Gold/Platinum
  * Progress bar to next tier
- Skill Points balance:
  * Large number
  * "Points History" link
- Swap statistics:
  * Total completed: 23
  * Success rate: 95%
  * Average rating: 4.8★

SKILLS SECTION:
- Skill cards with proficiency bars
- For each skill:
  * Icon + Name
  * Proficiency level badge
  * Years of experience
  * "Verified by 3 swaps" indicator
- "+ Add Skill" button

SWAP HISTORY TIMELINE:
- Recent 5 swaps
- "View All" button

ACTIONS:
- Edit Profile
- Settings
- Help & Support
- Logout

SETTINGS (Sub-screen):
- Language: English / ಕನ್ನಡ
- Notifications preferences
- Privacy settings
- Location sharing
- Data & Storage
- About & Terms

Design: Achievement-focused, gamified feel
Backend Requirements:

Kotlin

// Fetch user profile
db.collection("users")
    .document(currentUserId)
    .addSnapshotListener { doc, _ ->
        userProfile = doc?.toUserProfile()
    }

// Calculate Trust Score tier
fun calculateTier(trustScore: Int): Tier {
    return when (trustScore) {
        in 0..20 -> Tier.BRONZE
        in 21..50 -> Tier.SILVER
        in 51..80 -> Tier.GOLD
        else -> Tier.PLATINUM
    }
}

// Points history
db.collection("users")
    .document(currentUserId)
    .collection("transactions")
    .orderBy("timestamp", Query.Direction.DESCENDING)
    .get()
Connected To:

✅ Edit Profile (SCR-011A)
✅ Points History (SCR-011B)
✅ Settings (SCR-011C)
⬅️ Back to previous
📊 SCREEN NAVIGATION MAP
text

Splash (SCR-001)
    ├─→ Login (SCR-002)
    │       ├─→ OTP (SCR-003)
    │       │       ├─→ Profile Setup (SCR-004) [new users]
    │       │       └─→ Home (SCR-005) [returning users]
    │       └─→ Home (SCR-005)
    └─→ Home (SCR-005) [if authenticated]

Home (SCR-005) [MAIN HUB]
    ├─→ Post Need (SCR-007)
    │       └─→ Home (after posting)
    ├─→ Skill Board (SCR-006)
    │       └─→ Need Detail (SCR-008)
    │               ├─→ Make Offer (SCR-009)
    │               └─→ Poster Profile (SCR-011)
    ├─→ My Swaps (SCR-010)
    │       ├─→ Swap Detail (SCR-010B)
    │       ├─→ Chat (SCR-009)
    │       └─→ Dispute (SCR-010A)
    └─→ My Profile (SCR-011)
            ├─→ Edit Profile (SCR-011A)
            ├─→ Points History (SCR-011B)
            └─→ Settings (SCR-011C)

Bottom Navigation (Always Accessible):
├─ Home (SCR-005)
├─ Skill Board (SCR-006)
├─ My Swaps (SCR-010)
└─ Profile (SCR-011)
📄 PART 2: COMPREHENSIVE SOP DOCUMENT FOR DEVELOPMENT
STANDARD OPERATING PROCEDURE (SOP)
Skill-Exchange Android App Development
Document Version: 2.0
Prepared For: Antigravity Development Team
Project Code: VTU-SE-25
Date: January 2025

1. PROJECT OVERVIEW
1.1 Application Purpose
Skill-Exchange is a GenAI-powered Android application designed to enable cashless skill bartering among rural artisans in India. The app replaces informal, verbal skill exchanges with a tracked, equitable, trust-based digital platform.

1.2 Core Value Proposition
Problem Solved: Rural technicians need each other's skills but lack cash for payment
Solution: Digitized barter economy using Skill Points (1 hour = 1 point)
Differentiation: Trust Score system + GenAI assistance for low-literacy users
1.3 Target Users
Primary: Rural artisans (electricians, plumbers, carpenters, welders) in Karnataka
Age: 25-50 years
Literacy: Low to moderate
Tech: Basic smartphone users (WhatsApp proficient)
Income: ₹8,000-15,000/month, irregular
2. TECHNICAL ARCHITECTURE
2.1 Technology Stack
Layer	Technology	Version	Purpose
Platform	Android	SDK 26+ (Oreo+)	Mobile application
Language	Kotlin	1.9+	Primary development language
UI Framework	Jetpack Compose	Latest stable	Modern declarative UI
Design System	Material Design 3	Latest	UI components
Architecture	MVVM + Clean	-	Separation of concerns
Dependency Injection	Hilt (Dagger)	2.48+	DI framework
Async	Coroutines + Flow	Latest	Async operations
Authentication	Firebase Auth	Latest	Phone OTP
Database (Remote)	Cloud Firestore	Latest	Real-time data sync
Database (Local)	Room	2.6+	Offline cache
Storage	Firebase Cloud Storage	Latest	Images/files
Messaging	Firebase FCM	Latest	Push notifications
GenAI	Google Gemini API	Latest	AI features
Location	Google Play Services	Latest	Geolocation
Analytics	Firebase Analytics	Latest	Usage tracking
Crash Reporting	Firebase Crashlytics	Latest	Error monitoring
2.2 Architecture Layers
text

┌─────────────────────────────────────┐
│   PRESENTATION LAYER                │
│   - Jetpack Compose Screens         │
│   - ViewModels                      │
│   - UI State (StateFlow)            │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   DOMAIN LAYER                      │
│   - Use Cases                       │
│   - Business Logic                  │
│   - Repository Interfaces           │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   DATA LAYER                        │
│   - Repository Implementations      │
│   - Remote Data Source (Firestore)  │
│   - Local Data Source (Room)        │
│   - Sync Manager                    │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   EXTERNAL SERVICES                 │
│   - Firebase Services               │
│   - Gemini API                      │
│   - Google Play Services            │
└─────────────────────────────────────┘
3. DATA MODELS & FIRESTORE SCHEMA
3.1 Firestore Collections
Collection: users
Kotlin

data class UserProfile(
    val userId: String,                    // Document ID (Firebase UID)
    val name: String,
    val phoneNumber: String,
    val village: String,
    val taluka: String,
    val skills: List<Skill>,
    val trustScore: Int = 0,               // 0-100
    val skillPoints: Int = 0,
    val profilePhotoUrl: String? = null,
    val location: GeoPoint,
    val createdAt: Timestamp,
    val lastActive: Timestamp,
    val fcmToken: String? = null
)

data class Skill(
    val category: String,                  // from taxonomy
    val proficiency: String,               // Beginner/Intermediate/Expert
    val yearsExperience: Int,
    val verifiedSwaps: Int = 0
)
Security Rules:

JavaScript

match /users/{userId} {
  allow read: if request.auth != null;
  allow create: if request.auth.uid == userId;
  allow update: if request.auth.uid == userId;
  allow delete: if false; // No deletion allowed
}
Collection: needs
Kotlin

data class Need(
    val needId: String,
    val posterId: String,
    val title: String,
    val description: String,
    val skillRequired: String,
    val hoursNeeded: Int,
    val location: GeoPoint,
    val village: String,
    val urgency: String,                   // "urgent" or "flexible"
    val status: String,                    // "open", "matched", "closed"
    val photoUrl: String? = null,
    val timestamp: Timestamp,
    val offersCount: Int = 0
)
Security Rules:

JavaScript

match /needs/{needId} {
  allow read: if request.auth != null;
  allow create: if request.auth != null && 
                request.resource.data.posterId == request.auth.uid;
  allow update: if request.auth.uid == resource.data.posterId;
}
Collection: offers
Kotlin

data class Offer(
    val offerId: String,
    val needId: String,
    val offererId: String,
    val serviceOffered: String,
    val hoursOffered: Int,
    val status: String,                    // "pending", "accepted", "declined"
    val chatThread: List<Message>,
    val createdAt: Timestamp
)

data class Message(
    val senderId: String,
    val text: String,
    val timestamp: Timestamp,
    val type: String                       // "text", "voice", "image"
)
Collection: swaps
Kotlin

data class Swap(
    val swapId: String,
    val needId: String,
    val offerId: String,
    val partyA: String,                    // Need poster
    val partyB: String,                    // Offerer
    val serviceA: String,
    val serviceB: String,
    val hoursA: Int,
    val hoursB: Int,
    val confirmedByA: Boolean = false,
    val confirmedByB: Boolean = false,
    val status: String,                    // "active", "completed", "disputed"
    val createdAt: Timestamp,
    val completedAt: Timestamp? = null
)
Security Rules (Critical):

JavaScript

match /swaps/{swapId} {
  allow read: if request.auth.uid in [resource.data.partyA, resource.data.partyB];
  allow create: if request.auth != null;
  
  // Trust Score update only if BOTH parties confirmed
  allow update: if request.auth.uid in [resource.data.partyA, resource.data.partyB] &&
                (
                  (request.resource.data.confirmedByA == true && 
                   request.resource.data.confirmedByB == true) ||
                  request.auth.uid == resource.data.partyA ||
                  request.auth.uid == resource.data.partyB
                );
}
3.2 Room Database (Local)
Kotlin

@Database(
    entities = [
        NeedEntity::class,
        UserEntity::class,
        SwapEntity::class
    ],
    version = 1
)
abstract class SkillExchangeDatabase : RoomDatabase() {
    abstract fun needDao(): NeedDao
    abstract fun userDao(): UserDao
    abstract fun swapDao(): SwapDao
}

// Offline cache for Needs
@Entity(tableName = "needs")
data class NeedEntity(
    @PrimaryKey val needId: String,
    val title: String,
    val skillRequired: String,
    val cachedAt: Long
)
4. CORE FEATURES IMPLEMENTATION
4.1 Authentication Flow
Kotlin

// Phone Authentication
class AuthRepository @Inject constructor(
    private val auth: FirebaseAuth
) {
    fun sendOTP(phoneNumber: String): Flow<AuthState> = flow {
        val options = PhoneAuthOptions.newBuilder(auth)
            .setPhoneNumber("+91$phoneNumber")
            .setTimeout(60L, TimeUnit.SECONDS)
            .setCallbacks(object : PhoneAuthProvider.OnVerificationStateChangedCallbacks() {
                override fun onVerificationCompleted(credential: PhoneAuthCredential) {
                    emit(AuthState.Success(credential))
                }
                
                override fun onVerificationFailed(e: FirebaseException) {
                    emit(AuthState.Error(e.message))
                }
                
                override fun onCodeSent(
                    verificationId: String,
                    token: PhoneAuthProvider.ForceResendingToken
                ) {
                    emit(AuthState.CodeSent(verificationId))
                }
            })
            .build()
        
        PhoneAuthProvider.verifyPhoneNumber(options)
    }
    
    suspend fun verifyOTP(verificationId: String, code: String): AuthResult {
        val credential = PhoneAuthProvider.getCredential(verificationId, code)
        return auth.signInWithCredential(credential).await()
    }
}
4.2 Real-Time Need Board
Kotlin

class NeedRepository @Inject constructor(
    private val firestore: FirebaseFirestore,
    private val needDao: NeedDao
) {
    // Real-time listener
    fun getNeeds(filters: NeedFilters): Flow<List<Need>> = callbackFlow {
        var query: Query = firestore.collection("needs")
            .whereEqualTo("status", "open")
            .orderBy("timestamp", Query.Direction.DESCENDING)
        
        // Apply filters
        filters.skillCategory?.let {
            query = query.whereEqualTo("skillRequired", it)
        }
        
        val listener = query.addSnapshotListener { snapshot, error ->
            error?.let {
                trySend(emptyList())
                return@addSnapshotListener
            }
            
            val needs = snapshot?.documents?.mapNotNull { it.toNeed() } ?: emptyList()
            
            // Distance filtering (client-side)
            val filtered = filters.maxDistance?.let { maxKm ->
                needs.filter { need ->
                    calculateDistance(
                        filters.userLocation,
                        need.location
                    ) <= maxKm
                }
            } ?: needs
            
            // Cache locally
            needDao.insertAll(filtered.map { it.toEntity() })
            
            trySend(filtered)
        }
        
        awaitClose { listener.remove() }
    }
    
    // Offline fallback
    fun getCachedNeeds(): Flow<List<Need>> {
        return needDao.getCachedNeeds()
            .map { entities -> entities.map { it.toNeed() } }
    }
}
4.3 Trust Score System
Kotlin

class SwapRepository @Inject constructor(
    private val firestore: FirebaseFirestore
) {
    suspend fun confirmSwapCompletion(
        swapId: String,
        userId: String
    ): Result<Unit> = runCatching {
        firestore.runTransaction { transaction ->
            val swapRef = firestore.collection("swaps").document(swapId)
            val swap = transaction.get(swapRef).toSwap()
            
            // Determine which party is confirming
            val isPartyA = swap.partyA == userId
            val confirmField = if (isPartyA) "confirmedByA" else "confirmedByB"
            
            // Update confirmation
            transaction.update(swapRef, confirmField, true)
            
            // Check if BOTH parties confirmed
            val otherConfirmed = if (isPartyA) swap.confirmedByB else swap.confirmedByA
            
            if (otherConfirmed) {
                // BOTH confirmed - update Trust Scores
                val userARef = firestore.collection("users").document(swap.partyA)
                val userBRef = firestore.collection("users").document(swap.partyB)
                
                transaction.update(userARef, "trustScore", FieldValue.increment(1))
                transaction.update(userBRef, "trustScore", FieldValue.increment(1))
                
                transaction.update(userARef, "skillPoints", FieldValue.increment(swap.hoursA.toLong()))
                transaction.update(userBRef, "skillPoints", FieldValue.increment(swap.hoursB.toLong()))
                
                transaction.update(swapRef, mapOf(
                    "status" to "completed",
                    "completedAt" to FieldValue.serverTimestamp()
                ))
            }
        }.await()
    }
}
4.4 GenAI Integration
Kotlin

class GeminiService @Inject constructor() {
    private val gemini = GenerativeModel(
        modelName = "gemini-pro",
        apiKey = BuildConfig.GEMINI_API_KEY
    )
    
    // Voice-to-Text
    suspend fun transcribeVoice(audioBytes: ByteArray, language: String): String {
        // Implement using Google Speech-to-Text API
        // (Gemini doesn't directly support audio input yet)
        return speechToTextAPI.transcribe(audioBytes, language)
    }
    
    // Auto-tag skill from Need description
    suspend fun categorizeNeed(description: String): SkillCategory {
        val prompt = """
        Categorize this work request into ONE of these skill categories:
        - Electrical
        - Plumbing
        - Carpentry
        - Masonry
        - Welding
        - Agricultural Mechanics
        - General Labor
        - Tailoring
        
        Request: "$description"
        
        Respond with ONLY the category name.
        """
        
        val response = gemini.generateContent(prompt).text
        return parseSkillCategory(response)
    }
    
    // Estimate hours needed
    suspend fun estimateHours(description: String, skillCategory: String): Int {
        val prompt = """
        Estimate hours needed for this $skillCategory task:
        "$description"
        
        Respond with ONLY a number (1-24).
        """
        
        val response = gemini.generateContent(prompt).text
        return response.trim().toIntOrNull() ?: 2 // Default 2 hours
    }
    
    // Fairness analysis
    suspend fun analyzeFairness(swap: SwapProposal): FairnessAnalysis {
        val prompt = """
        Analyze this skill swap:
        
        Person A offers: ${swap.hoursA} hours of ${swap.skillA}
        Person B offers: ${swap.hoursB} hours of ${swap.skillB}
        
        Is this fair? Consider:
        1. Skill complexity
        2. Physical demands
        3. Material costs
        4. Market rates in rural India
        
        Respond in this format:
        Fair: YES/NO
        Reason: [brief explanation in simple language]
        Suggestion: [if not fair, suggest adjustment]
        """
        
        val response = gemini.generateContent(prompt).text
        return parseFairnessAnalysis(response)
    }
    
    // Recommended Needs (matching)
    suspend fun matchNeeds(userSkills: List<String>, availableNeeds: List<Need>): List<Need> {
        val skillsText = userSkills.joinToString()
        val needsText = availableNeeds.take(20).map { 
            "${it.needId}: ${it.skillRequired} - ${it.title}"
        }.joinToString("\n")
        
        val prompt = """
        User has these skills: $skillsText
        
        Available needs:
        $needsText
        
        Return the top 5 need IDs that best match the user's skills, comma-separated.
        """
        
        val response = gemini.generateContent(prompt).text
        val matchedIds = response.split(",").map { it.trim() }
        
        return availableNeeds.filter { it.needId in matchedIds }
    }
}
4.5 Offline Support
Kotlin

class SyncManager @Inject constructor(
    private val firestore: FirebaseFirestore,
    private val needDao: NeedDao,
    private val connectivityObserver: ConnectivityObserver
) {
    init {
        // Monitor connectivity
        connectivityObserver.observe().onEach { isConnected ->
            if (isConnected) {
                syncPendingData()
            }
        }.launchIn(CoroutineScope(Dispatchers.IO))
    }
    
    private suspend fun syncPendingData() {
        // Sync draft Needs
        val draftNeeds = needDao.getDrafts()
        draftNeeds.forEach { draft ->
            try {
                firestore.collection("needs").add(draft.toMap()).await()
                needDao.delete(draft)
            } catch (e: Exception) {
                Timber.e(e, "Failed to sync draft Need")
            }
        }
        
        // Sync pending confirmations
        // ... similar logic
    }
    
    // Enable Firestore offline persistence
    fun enableOfflineMode() {
        firestore.firestoreSettings = firestoreSettings {
            isPersistenceEnabled = true
            cacheSizeBytes = FirebaseFirestoreSettings.CACHE_SIZE_UNLIMITED
        }
    }
}
5. UI/UX IMPLEMENTATION GUIDELINES
5.1 Design Tokens
Kotlin

// Color Palette
object SkillExchangeColors {
    val Terracotta = Color(0xFFE07856)
    val Saffron = Color(0xFFFF9933)
    val LeafGreen = Color(0xFF4CAF50)
    val EarthBrown = Color(0xFF8D6E63)
    val Cream = Color(0xFFFFF8E1)
    val DarkGray = Color(0xFF424242)
    val LightGray = Color(0xFFEEEEEE)
}

// Typography
val SkillExchangeTypography = Typography(
    displayLarge = TextStyle(
        fontSize = 32.sp,
        fontWeight = FontWeight.Bold,
        color = DarkGray
    ),
    bodyLarge = TextStyle(
        fontSize = 18.sp,
        fontWeight = FontWeight.Normal,
        lineHeight = 24.sp
    ),
    // Minimum 16sp for accessibility
    bodyMedium = TextStyle(
        fontSize = 16.sp
    )
)

// Spacing
object Spacing {
    val xs = 4.dp
    val s = 8.dp
    val m = 16.dp
    val l = 24.dp
    val xl = 32.dp
}
5.2 Accessibility Requirements
Kotlin

@Composable
fun AccessibleButton(
    text: String,
    onClick: () -> Unit,
    modifier: Modifier = Modifier
) {
    Button(
        onClick = onClick,
        modifier = modifier
            .heightIn(min = 48.dp) // Minimum touch target
            .semantics {
                contentDescription = text
                role = Role.Button
            },
        colors = ButtonDefaults.buttonColors(
            containerColor = Terracotta,
            contentColor = Color.White
        )
    ) {
        Text(
            text = text,
            fontSize = 18.sp,
            fontWeight = FontWeight.SemiBold
        )
    }
}

// Screen reader support
@Composable
fun TrustScoreBadge(score: Int) {
    Box(
        modifier = Modifier.semantics {
            contentDescription = "Trust Score: $score out of 100"
            stateDescription = getTierName(score)
        }
    ) {
        // Visual badge
        BadgeContent(score)
    }
}
5.3 Voice Input Component
Kotlin

@Composable
fun VoiceInputField(
    onTranscriptionComplete: (String) -> Unit,
    language: String = "kn" // Kannada
) {
    var isRecording by remember { mutableStateOf(false) }
    var transcription by remember { mutableStateOf("") }
    
    val speechRecognizer = rememberSpeechRecognizer(
        language = language,
        onResult = { result ->
            transcription = result
            onTranscriptionComplete(result)
            isRecording = false
        }
    )
    
    Column(horizontalAlignment = Alignment.CenterHorizontally) {
        IconButton(
            onClick = {
                if (isRecording) {
                    speechRecognizer.stop()
                } else {
                    speechRecognizer.start()
                }
                isRecording = !isRecording
            },
            modifier = Modifier.size(80.dp)
        ) {
            Icon(
                imageVector = if (isRecording) Icons.Filled.Stop else Icons.Filled.Mic,
                contentDescription = if (isRecording) "Stop recording" else "Start recording",
                tint = if (isRecording) Color.Red else Terracotta,
                modifier = Modifier.size(48.dp)
            )
        }
        
        if (isRecording) {
            WaveformAnimation()
        }
        
        if (transcription.isNotEmpty()) {
            Text(
                text = transcription,
                modifier = Modifier.padding(top = 16.dp),
                fontSize = 16.sp
            )
        }
    }
}
6. CRITICAL BUSINESS LOGIC RULES
6.1 Trust Score Update Rules
✅ MUST ENFORCE:

Trust Score increments ONLY when BOTH parties independently confirm swap completion
Single-party confirmation does NOT increase score
72-hour window for dual confirmation
After 72 hours with single confirmation → "Disputed" status → Moderator review
Trust Score can only increase, never decrease (maintains user trust)
Score range: 0-100
Implementation Checklist:

 Firestore transaction for atomic dual-confirmation check
 Security rules prevent unauthorized score manipulation
 Scheduled function to check 72-hour disputes
 Moderator dashboard for dispute resolution
6.2 Skill Point Economics
✅ MUST ENFORCE:

1 hour of skilled labor = 1 Skill Point
Points awarded only after swap completion (both confirmations)
Points are non-transferable between users
Points do NOT expire (v1.0 - may change in v2.0)
Negative balances NOT allowed
6.3 Need Board Filtering
✅ MUST IMPLEMENT:

Real-time Firestore query with compound indexes
Filter by Skill Category (dropdown selection)
Filter by Distance (5km / 10km / 25km / Any)
Filter by Urgency (Urgent / Flexible)
Filter by Status (Open / Matched / Closed)
Firestore Composite Index Required:

text

Collection: needs
Fields:
  - status (Ascending)
  - skillRequired (Ascending)
  - timestamp (Descending)
7. TESTING REQUIREMENTS
7.1 Unit Tests (Minimum 60% Coverage)
Kotlin

class TrustScoreUseCaseTest {
    @Test
    fun `confirmSwap updates score only when both parties confirm`() = runTest {
        // Given
        val swap = Swap(
            swapId = "test123",
            partyA = "userA",
            partyB = "userB",
            confirmedByA = true,
            confirmedByB = false,
            status = "active"
        )
        
        // When
        swapRepository.confirmCompletion("test123", "userB")
        
        // Then
        val updatedSwap = swapRepository.getSwap("test123")
        assertEquals("completed", updatedSwap.status)
        
        val userA = userRepository.getUser("userA")
        assertEquals(1, userA.trustScore) // Increased
    }
}
7.2 Integration Tests
Kotlin

@HiltAndroidTest
class NeedBoardIntegrationTest {
    @Test
    fun `filter by skill category returns correct results`() {
        // Test Firestore query filtering
    }
    
    @Test
    fun `offline mode loads cached needs`() {
        // Test Room DB fallback
    }
}
7.3 UI Tests
Kotlin

@Test
fun testNeedPostingFlow() {
    composeTestRule.apply {
        onNodeWithText("+ Post Need").performClick()
        onNodeWithContentDescription("Voice input").performClick()
        // Simulate voice transcription
        onNodeWithText("Post Need").performClick()
        
        // Verify Need appears in feed
        onNodeWithText("Your need title").assertExists()
    }
}
7.4 User Acceptance Testing (UAT)
Required before launch:

 5 rural artisan participants
 Age range: 30-50
 Low to moderate literacy
 Test in real village settings (2G network)
 Minimum 4/5 usability rating
 No critical bugs reported
UAT Script:

Register new account with phone number
Create profile with 2 skills
Post a Need using voice input
Browse Skill Board and filter by category
Make a Swap Offer
Complete a swap and confirm
Check Trust Score increase
8. DEPLOYMENT & RELEASE
8.1 Build Configuration
gradle

// app/build.gradle.kts
android {
    defaultConfig {
        applicationId = "com.mindmatrix.skillexchange"
        minSdk = 26
        targetSdk = 34
        versionCode = 1
        versionName = "1.0.0"
    }
    
    buildTypes {
        release {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
            signingConfig = signingConfigs.getByName("release")
        }
    }
    
    bundle {
        language {
            enableSplit = false // Include all languages
        }
        density {
            enableSplit = true
        }
        abi {
            enableSplit = true
        }
    }
}
8.2 Continuous Integration
YAML

# .github/workflows/android.yml
name: Android CI
on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Build with Gradle
        run: ./gradlew build
      - name: Run tests
        run: ./gradlew test
      - name: Upload APK
        uses: actions/upload-artifact@v3
        with:
          name: app-release.apk
          path: app/build/outputs/apk/release/
8.3 Google Play Store Release Checklist
 App signing key generated and backed up
 Privacy Policy URL created
 App screenshots (8 required: 5 phone, 3 tablet)
 Feature graphic (1024 x 500)
 App icon (512 x 512)
 Short description (80 chars max)
 Full description (4000 chars max)
 Content rating questionnaire completed
 Target age group: Adults
 Category: Productivity / Business
 Closed beta test with 20 users
 Open beta test with 100 users
 Production release
9. MONITORING & ANALYTICS
9.1 Firebase Analytics Events
Kotlin

// Track key user actions
fun logNeedPosted(skillCategory: String) {
    analytics.logEvent("need_posted") {
        param("skill_category", skillCategory)
        param("method", "voice") // or "text"
    }
}

fun logSwapCompleted(skillA: String, skillB: String) {
    analytics.logEvent("swap_completed") {
        param("skill_a", skillA)
        param("skill_b", skillB)
    }
}

fun logTrustScoreIncrease(oldScore: Int, newScore: Int) {
    analytics.logEvent("trust_score_increase") {
        param("old_score", oldScore.toLong())
        param("new_score", newScore.toLong())
    }
}
9.2 Performance Monitoring
Kotlin

// Track screen load times
val trace = Firebase.performance.newTrace("need_board_load")
trace.start()
// Load data
trace.stop()

// Network request monitoring (automatic with Firebase Performance)
9.3 Success Metrics Dashboard
Weekly Review Metrics:

Metric	Target	Actual
Active Users	100	-
Completed Swaps	20	-
Skill Points Transacted	500	-
Average Trust Score	10+	-
App Crash Rate	<1%	-
User Retention (D30)	40%	-
10. SECURITY & PRIVACY
10.1 Data Protection
Personal Data Collected:

Phone number (for authentication)
Name, village/taluka (for profile)
GPS location (with explicit permission)
Skill information
Swap history
Data NOT Collected:

Aadhaar / Government IDs
Financial information
Device contacts
Call logs
10.2 Security Measures
Kotlin

// Encrypt sensitive local data
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()

val encryptedPrefs = EncryptedSharedPreferences.create(
    context,
    "secure_prefs",
    masterKey,
    EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
    EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
)
Firestore Security Rules (Comprehensive):

JavaScript

rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    
    // Users
    match /users/{userId} {
      allow read: if request.auth != null;
      allow create: if request.auth.uid == userId;
      allow update: if request.auth.uid == userId &&
                    // Prevent self-manipulation of trustScore
                    request.resource.data.trustScore == resource.data.trustScore;
    }
    
    // Needs
    match /needs/{needId} {
      allow read: if request.auth != null;
      allow create: if request.auth != null &&
                    request.resource.data.posterId == request.auth.uid;
      allow update: if request.auth.uid == resource.data.posterId;
      allow delete: if request.auth.uid == resource.data.posterId &&
                    resource.data.status == 'open'; // Can only delete open Needs
    }
    
    // Swaps (Critical)
    match /swaps/{swapId} {
      allow read: if request.auth.uid in [resource.data.partyA, resource.data.partyB];
      allow create: if request.auth != null;
      allow update: if request.auth.uid in [resource.data.partyA, resource.data.partyB] &&
                    // Prevent status manipulation
                    (request.resource.data.status == resource.data.status ||
                     (request.resource.data.confirmedByA && request.resource.data.confirmedByB));
    }
  }
}
11. PROJECT DELIVERABLES
11.1 Code Repository Structure
text

skill-exchange-android/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/mindmatrix/skillexchange/
│   │   │   │   ├── data/
│   │   │   │   │   ├── local/      # Room DB
│   │   │   │   │   ├── remote/     # Firestore
│   │   │   │   │   └── repository/
│   │   │   │   ├── domain/
│   │   │   │   │   ├── model/
│   │   │   │   │   └── usecase/
│   │   │   │   ├── presentation/
│   │   │   │   │   ├── auth/
│   │   │   │   │   ├── home/
│   │   │   │   │   ├── needboard/
│   │   │   │   │   ├── swap/
│   │   │   │   │   └── profile/
│   │   │   │   ├── ui/
│   │   │   │   │   ├── components/
│   │   │   │   │   └── theme/
│   │   │   │   └── util/
│   │   │   └── res/
│   │   └── test/
│   └── build.gradle.kts
├── gradle/
├── README.md
├── LICENSE
└── .gitignore
11.2 Documentation
Required Files:

README.md - Setup instructions
API_DOCUMENTATION.md - Firestore structure, API endpoints
ARCHITECTURE.md - Technical architecture diagram
TESTING_GUIDE.md - How to run tests
DEPLOYMENT_GUIDE.md - Release process
PRIVACY_POLICY.md - User-facing privacy policy
CHANGELOG.md - Version history
11.3 Final Submission Package
Deliverables to MindMatrix:

 GitHub repository link (public or private with access)
 Signed APK file (app-release.apk)
 Demo video (5 minutes max)
 Presentation deck (PPT/PDF)
 User testing report (5 participants)
 Firebase project export (Firestore rules, indexes)
 API keys documentation (securely shared)
 Play Store listing materials (screenshots, descriptions)
12. TIMELINE & MILESTONES
Week	Milestone	Deliverables	Status
1-2	Foundation	Firebase setup, Auth, User Profile	🔄
3-4	Core Loop	Need Board, Post Need, Filters	⏳
5-6	Trust Layer	Swap Offers, Chat, Dual Confirmation	⏳
7	AI Layer	GenAI voice input, matching, fairness	⏳
8	Hardening	Offline mode, optimization, accessibility	⏳
9	Validation	UAT with 5 rural users, bug fixes	⏳
10	Delivery	Final APK, documentation, presentation	⏳
13. SUPPORT & MAINTENANCE
13.1 Post-Launch Support Plan
First 30 Days:

Daily monitoring of Crashlytics
24-hour response time for critical bugs
Weekly analytics review
User feedback collection via in-app form
Ongoing:

Monthly security updates
Quarterly feature additions
Bi-annual dependency updates
13.2 Known Limitations (v1.0)
Out of Scope:

Multi-language beyond Kannada/English
Monetary transactions (UPI/wallets)
iOS version
Web dashboard
Government integrations
Planned for v2.0:

Skill verification badges
Community leaderboards
Multi-village federations
Advanced dispute resolution AI
Skill training recommendations
END OF SOP DOCUMENT
📎 APPENDIX A: QUICK REFERENCE
Key Commands
Bash

# Build APK
./gradlew assembleRelease

# Run tests
./gradlew test

# Deploy Firestore rules
firebase deploy --only firestore:rules

# Deploy Firestore indexes
firebase deploy --only firestore:indexes

# Generate signed bundle
./gradlew bundleRelease
Emergency Contacts
Firebase Support: firebase.google.com/support
Gemini API Issues: ai.google.dev/support
MindMatrix Coordinator: [Contact Email]
