# SarkarSahayak - Requirements Document

## Project Overview
**SarkarSahayak** (Government Helper) is an AI-powered multilingual chatbot that helps Indian citizens discover and access government schemes they are eligible for, breaking down barriers of language, literacy, and digital complexity.

---

## Problem Statement
Track 03: AI for Communities, Access & Public Impact

Build an AI-powered solution that improves access to information, resources, or opportunities for communities and public systems.

---

## Target Audience
- Rural and semi-urban citizens (Tier 2/3/4 cities and villages)
- Low-income families earning less than ₹5 lakh annually
- Farmers, pregnant women, senior citizens, small business owners
- People with limited digital literacy
- Non-English speakers (700M+ Indians)

---

## Core Problem Analysis

### Current Challenges:
1. **Information Gap**: 65% of eligible citizens are unaware of government schemes
2. **Language Barrier**: Most government portals are English-only
3. **Complexity**: Difficult navigation, technical jargon, confusing eligibility criteria
4. **Digital Divide**: Complex forms deter people with low digital literacy
5. **Time Wastage**: Citizens visit multiple offices only to find they're ineligible
6. **Missed Benefits**: Billions in allocated funds remain unclaimed annually

### Real-World Impact:
- Farmers miss PM-KISAN (₹6000/year)
- Pregnant women miss PMMVY (₹5000 maternity benefit)
- Families miss Ayushman Bharat (₹5 lakh health cover)
- Girls miss scholarship opportunities
- Small businesses miss Mudra loans

---

## Functional Requirements

### FR1: Multilingual Conversational Interface
- **FR1.1**: Support for Hindi, English, Tamil, Bengali, Marathi, Telugu, Gujarati
- **FR1.2**: Natural language processing to understand user queries in regional languages
- **FR1.3**: Voice input and output capabilities for low-literacy users
- **FR1.4**: Simple chat-based interface requiring no technical knowledge

### FR2: Intelligent Scheme Discovery
- **FR2.1**: Ask contextual questions to understand user profile (age, income, occupation, state, family details)
- **FR2.2**: Match user profile against 50+ central and state government schemes
- **FR2.3**: Rank schemes by relevance and benefit amount
- **FR2.4**: Explain eligibility criteria in simple, jargon-free language

### FR3: Personalized Recommendations
- **FR3.1**: Generate customized scheme recommendations based on user data
- **FR3.2**: Provide scheme details including:
  - Benefit amount and type
  - Eligibility requirements
  - Required documents
  - Application deadline
  - Offline and online application process
- **FR3.3**: Multi-scheme eligibility (show all applicable schemes, not just one)

### FR4: Application Assistance
- **FR4.1**: Step-by-step guidance for filling application forms
- **FR4.2**: Document checklist generation
- **FR4.3**: Direct links to official application portals
- **FR4.4**: WhatsApp/SMS integration for form links and reminders

### FR5: Knowledge Base Management
- **FR5.1**: Comprehensive database of central government schemes
- **FR5.2**: State-specific schemes (starting with 5 major states)
- **FR5.3**: Regular updates when new schemes launch or eligibility changes
- **FR5.4**: District-level scheme variations

### FR6: User Interaction Features
- **FR6.1**: Save user profile for future visits (optional, with consent)
- **FR6.2**: Share scheme information via WhatsApp/SMS
- **FR6.3**: Bookmark favorite schemes
- **FR6.4**: FAQ section for common questions

### FR7: Accessibility Features
- **FR7.1**: Text-to-speech for illiterate users
- **FR7.2**: Large font options for senior citizens
- **FR7.3**: Simple color-coded UI (green = eligible, red = not eligible)
- **FR7.4**: Works on low-bandwidth connections (2G/3G)

---

## Non-Functional Requirements

### NFR1: Performance
- **NFR1.1**: Response time < 3 seconds for scheme recommendations
- **NFR1.2**: Support 1000+ concurrent users
- **NFR1.3**: 99.5% uptime during peak hours (9 AM - 6 PM IST)
- **NFR1.4**: Lightweight design for low-end smartphones (< 5MB data per session)

### NFR2: Scalability
- **NFR2.1**: Horizontal scaling to handle 10,000+ daily active users
- **NFR2.2**: Database architecture supporting 200+ schemes across 28 states
- **NFR2.3**: Modular language support (add new languages without code changes)
- **NFR2.4**: API-first design for future integrations (UMANG app, DigiLocker)

### NFR3: Security & Privacy
- **NFR3.1**: End-to-end encryption for personal data
- **NFR3.2**: No mandatory user registration or data collection
- **NFR3.3**: GDPR and Indian IT Act 2000 compliance
- **NFR3.4**: Anonymous usage analytics (no PII stored)
- **NFR3.5**: Option to delete conversation history

### NFR4: Usability
- **NFR4.1**: Zero learning curve - anyone can use in < 2 minutes
- **NFR4.2**: Maximum 5 questions to get scheme recommendations
- **NFR4.3**: Mobile-first responsive design
- **NFR4.4**: Works without app installation (web-based + PWA)

### NFR5: Reliability
- **NFR5.1**: Accurate scheme eligibility matching (95%+ accuracy)
- **NFR5.2**: Graceful degradation if AI service is down (fallback to rule-based matching)
- **NFR5.3**: Regular data validation against official government sources
- **NFR5.4**: Error handling with helpful messages in user's language

### NFR6: Maintainability
- **NFR6.1**: Modular codebase for easy updates
- **NFR6.2**: Automated scheme data ingestion from government APIs
- **NFR6.3**: Admin dashboard for content updates (non-technical staff can update)
- **NFR6.4**: Comprehensive logging for debugging

### NFR7: Compatibility
- **NFR7.1**: Works on Android 5.0+, iOS 12+
- **NFR7.2**: Browser support: Chrome, Firefox, Safari, UC Browser
- **NFR7.3**: Progressive Web App (PWA) for offline access
- **NFR7.4**: WhatsApp Business API integration

### NFR8: Localization
- **NFR8.1**: Cultural sensitivity in language and examples
- **NFR8.2**: Regional festivals and events consideration
- **NFR8.3**: State-specific terminology (e.g., "Ration Card" vs "Food Security Card")

---

## AI/ML Requirements

### AI1: Natural Language Understanding
- Google Gemini Pro API for conversational AI
- Intent classification (greeting, scheme_query, eligibility_check, application_help)
- Entity extraction (age, income, occupation, state, category)
- Sentiment analysis for user satisfaction

### AI2: Scheme Matching Algorithm
- Rule-based filtering for hard eligibility criteria (age, income thresholds)
- Semantic similarity matching for soft criteria
- Confidence scoring for each recommendation
- Explainable AI - show WHY user is eligible

### AI3: Multilingual NLP
- Translation layer using Google Translate API
- Language detection (auto-detect user's preferred language)
- Transliteration support (Roman script for Indic languages)
- Context-aware translation (preserve scheme names in original language)

### AI4: Continuous Learning
- User feedback loop to improve recommendations
- A/B testing for conversation flows
- Analytics on most-searched schemes, common drop-off points

---

## Data Requirements

### Government Schemes Database
**Central Schemes (Minimum 30):**
- PM-KISAN (Agriculture)
- Ayushman Bharat (Healthcare)
- PM Awas Yojana (Housing)
- PMMY - Mudra Loans (Business)
- Sukanya Samriddhi Yojana (Girl Child)
- PM Matru Vandana Yojana (Pregnant Women)
- Atal Pension Yojana (Senior Citizens)
- PM Scholarship Scheme (Students)
- And 22 more...

**State Schemes (Starting 5 States):**
- Maharashtra, Tamil Nadu, Karnataka, Uttar Pradesh, West Bengal
- 4 schemes per state = 20 additional schemes

**Data Fields per Scheme:**
- Scheme ID, Name (Hindi + English), Category
- Eligibility criteria (structured JSON)
- Benefits (amount, type, duration)
- Required documents list
- Application process (online/offline)
- Official portal link
- Helpline number
- Last updated timestamp

### User Profile Data (Optional Storage)
- Age, Gender, State, District
- Occupation, Annual Income
- Family details (optional)
- Language preference
- Stored only with explicit consent

---

## Integration Requirements

### INT1: Government APIs
- MyGov API for scheme updates (if available)
- UMANG App integration for seamless application
- DigiLocker for document verification (future scope)

### INT2: Communication Channels
- WhatsApp Business API for notifications
- SMS gateway for non-smartphone users
- Web interface (primary)
- Mobile app (future scope)

### INT3: Payment & Authentication
- No payment required (free public service)
- Optional Aadhaar-based profile (for personalization, not mandatory)

---

## Success Metrics

### User Engagement
- 10,000+ unique users in first month
- 60%+ users complete full eligibility check
- Average 3+ schemes discovered per user
- 40%+ users click "Apply Now" link

### Impact Metrics
- 5000+ successful scheme applications attributed to platform
- Coverage across 15+ states
- 70%+ user satisfaction score
- 50%+ users in Tier 2/3/4 cities

### Technical Metrics
- 95%+ scheme recommendation accuracy
- <3 second average response time
- <1% error rate
- 99%+ uptime

---

## Future Enhancements (Post-Hackathon)

### Phase 1 Enhancements (Months 1-3)
1. **Application Tracking**: Track scheme application status end-to-end
2. **Document Upload**: Help users digitize documents using phone camera + OCR
3. **Video Tutorials**: Explain schemes via short videos in regional languages
4. **Offline Mode**: Download scheme database for offline access via PWA

### Phase 2 Enhancements (Months 4-6)
5. **Agent Network**: Partner with Common Service Centers (CSCs) for assisted access
6. **Blockchain Verification**: Tamper-proof scheme eligibility certificates
7. **AI-Powered Form Filling**: Auto-fill application forms using user data

### Phase 3: Integrated Civic Platform (Months 7-12)

Building on SarkarSahayak's success, we'll expand into a comprehensive AI-powered civic engagement platform:

#### **3.1 NagrikVaani - AI Complaint Management Module**

**Purpose**: Citizens who discover schemes through SarkarSahayak can also report issues with implementation

**Features**:
- **Smart Categorization**: AI auto-assigns complaints to correct government department
  - "PM-KISAN payment not received" → Ministry of Agriculture
  - "Ayushman Bharat card not working" → Health Department
- **Duplicate Detection**: Groups similar complaints to prevent resource wastage
  - If 50 people report "Ration shop closed in Area X", cluster them together
- **Priority Scoring**: Urgent safety issues flagged immediately
  - Hospital access blocked > Street light broken
- **Sentiment Analysis**: Detects frustrated citizens for priority handling
- **Image Verification**: AI validates complaint photos to prevent fraud
- **Status Updates**: Auto-notify complainants when action is taken
- **Resolution Prediction**: "Based on similar cases, expected resolution in 7 days"

**Integration with SarkarSahayak**:
- Users discover schemes → Apply → If issue during application → File complaint seamlessly
- Single platform for discovery AND accountability
- Government gets feedback loop on scheme implementation issues

**Technical Implementation**:
```
Complaint Input (Text + Image + Location)
    ↓
NLP Classification (Category, Department, Priority)
    ↓
Sentiment Analysis (Urgent/Angry/Frustrated)
    ↓
Duplicate Detection (Clustering algorithm)
    ↓
Image Verification (Google Vision API)
    ↓
Auto-assign to Department + Send SMS/Email
    ↓
Track Resolution + Notify Citizen
```

**Impact Metrics**:
- Reduce complaint resolution time from 45 days → 7 days
- 80% auto-categorization accuracy
- Prevent duplicate handling of 40% complaints
- Citizen satisfaction increase from 30% → 75%

---

#### **3.2 JantaKaPoll - Public Opinion Mining Module**

**Purpose**: Government launches new schemes; SarkarSahayak measures real-time public sentiment

**Features**:
- **Real-time Sentiment Dashboard**: 
  - Monitor Twitter, Facebook, Reddit, YouTube comments
  - "PM Awas Yojana: 65% positive sentiment, trending concerns: slow approval process"
- **Multilingual Opinion Analysis**: 
  - Analyze Hindi tweets, Tamil posts, Bengali comments simultaneously
  - Detect regional differences in scheme perception
- **Trend Detection**:
  - "Unemployment scheme sentiment dropped 20% this week - investigate why"
  - Early warning system for policy issues BEFORE they become protests
- **Topic Modeling**:
  - What are people discussing about education schemes?
  - Top concerns: Quality of schools (35%), Teacher shortage (28%), Mid-day meal issues (15%)
- **Demographic Insights**:
  - Farmers love PM-KISAN but want faster disbursement
  - Urban youth want more skill development schemes
- **Scheme Comparison**:
  - "Which healthcare scheme is more popular: Ayushman Bharat vs State schemes?"

**Integration with SarkarSahayak**:
- User feedback from chatbot conversations feeds into sentiment analysis
- "5000 users asked about PM-KISAN this week but 40% expressed confusion about eligibility"
- Government can improve scheme communication based on real data

**Technical Implementation**:
```
Data Sources: Twitter API + Reddit + News Comments + In-app Feedback
    ↓
Text Preprocessing (Remove spam, translate to English)
    ↓
Sentiment Classification (Positive/Negative/Neutral per scheme)
    ↓
Topic Modeling (LDA/BERT) - Extract key themes
    ↓
Trend Detection (Time-series analysis)
    ↓
Dashboard Visualization for Policymakers
```

**Use Cases**:
1. **Budget Reaction**: Analyze 1M social media posts in 10 minutes after budget announcement
2. **Scheme Launch**: Monitor public reception of new scheme in real-time
3. **Crisis Prevention**: Detect growing negative sentiment before it becomes a protest
4. **Policy Improvement**: Identify specific pain points in scheme implementation

**Impact Metrics**:
- Analyze 100K+ social media posts daily
- Real-time sentiment updates (refresh every 15 minutes)
- Regional sentiment breakdown (28 states)
- 85% accuracy in sentiment classification

---

#### **3.3 CrisisConnect - Emergency Response Integration**

**Purpose**: During disasters, SarkarSahayak becomes emergency assistance platform

**Features**:
- **Disaster Scheme Discovery**:
  - Floods in Kerala → Auto-recommend NDRF relief schemes, insurance claims
  - Cyclone alert → Show evacuation centers, emergency cash schemes
- **Real-time Emergency Alerts**:
  - "Heavy rainfall predicted in your area. Eligible for PM Fasal Bima Yojana crop insurance"
  - "Cyclone warning: Nearest shelter at XYZ location. Click for evacuation route"
- **SOS Integration**:
  - Users can report emergencies through chatbot
  - "Need medical help, flood water rising" → Auto-alert rescue teams
  - Verify authenticity using AI (fake vs real emergency)
- **Resource Mapping**:
  - Show nearest relief camps, food distribution centers, medical facilities
  - Real-time availability updates
- **Multi-channel Alerts**:
  - WhatsApp messages (works on 2G)
  - SMS alerts (no internet needed)
  - In-app notifications
- **Post-disaster Scheme Access**:
  - "Your house was damaged? Check PM Awas Yojana reconstruction schemes"
  - Fast-track applications during crisis

**Integration with SarkarSahayak**:
- Normal times: Scheme discovery platform
- Emergency: Transforms into disaster response system
- Post-disaster: Helps victims access relief schemes

**Technical Implementation**:
```
Monitor Weather APIs + Disaster Management Databases
    ↓
Detect Crisis in User's Location
    ↓
Send Proactive Alerts (SMS/WhatsApp)
    ↓
Switch UI to Emergency Mode
    ↓
Show Relevant Relief Schemes + Resources
    ↓
Accept SOS Requests + Verify Authenticity
    ↓
Coordinate with Disaster Response Teams
```

**Disaster Coverage**:
- Floods, cyclones, earthquakes, droughts
- Industrial accidents, epidemics
- Man-made disasters

**Impact Metrics**:
- Alert 1M+ people in disaster-prone areas
- 10-minute response time for emergency alerts
- 95% delivery rate (multi-channel redundancy)
- Coordinate 500+ rescue operations during major disaster

---

### Unified Platform Vision

**SarkarSahayak Evolution**:
```
Phase 1: Scheme Discovery (Current)
    ↓
Phase 2: Scheme + Complaints (NagrikVaani)
    ↓
Phase 3: + Opinion Mining (JantaKaPoll)
    ↓
Phase 4: + Emergency Response (CrisisConnect)
    ↓
Final Vision: Complete Civic Engagement Platform
```

**One App for Everything**:
- 🎯 Discover schemes you're eligible for
- 📢 File complaints about implementation
- 🗳️ Share feedback on government policies
- 🚨 Get emergency alerts and assistance
- 📊 Government gets real-time citizen insights

**Why This Integration Works**:
1. **Single User Base**: Build trust with scheme discovery, extend to other services
2. **Data Synergy**: Complaint data improves scheme recommendations, sentiment data guides policy
3. **Network Effects**: More users = better AI = more value
4. **Government Buy-in**: One platform to manage, cheaper than separate systems

**Technical Synergies**:
- Shared AI infrastructure (Gemini API for all modules)
- Common user authentication and profiles
- Unified database (schemes, complaints, sentiment data)
- Single mobile app/web interface

**Impact at Scale**:
- **10 Million active users** by Year 2
- **₹10,000 crores** in benefits claimed through platform
- **50% reduction** in complaint resolution time
- **1000+ lives saved** through emergency alerts
- **Real-time feedback loop** between citizens and government

---

## Constraints & Assumptions

### Assumptions:
1. Users have access to basic smartphone or feature phone with internet
2. Government scheme data is publicly available and structured
3. Users are willing to share basic demographic information
4. Internet penetration continues to grow in rural India

### Constraints:
1. Initial version supports 7 languages (can expand based on demand)
2. Limited to central + 5 state schemes in MVP
3. Dependent on third-party AI APIs (Google Gemini)
4. No offline application submission (only guidance)

### Risks:
1. **Data Accuracy**: Government schemes change frequently
   - *Mitigation*: Automated weekly data sync + manual review
2. **User Trust**: People may doubt AI recommendations
   - *Mitigation*: Show official government links, add disclaimer
3. **Digital Literacy**: Very low literacy users may struggle
   - *Mitigation*: Voice interface, video tutorials, CSC partnerships
4. **Scalability Costs**: High usage may exceed free API limits
   - *Mitigation*: Caching, rate limiting, apply for government grants

---

## Compliance & Legal

- **Data Protection**: Follow IT Act 2000, Digital Personal Data Protection Act 2023
- **Disclaimer**: Not an official government service, recommendations are indicative
- **Accessibility**: WCAG 2.1 Level AA compliance for persons with disabilities
- **Content**: All scheme information sourced from official government portals

---

## Conclusion

SarkarSahayak addresses a critical gap in India's digital governance ecosystem. By combining AI, multilingual support, and user-centric design, it empowers millions of citizens to access their rightful government benefits.

**Current MVP Impact**: If just 1% of eligible citizens (9 million people) use SarkarSahayak and claim one scheme worth ₹5000 on average, that's ₹4,500 crores reaching the intended beneficiaries.

**Future Platform Vision**: By integrating NagrikVaani (complaints), JantaKaPoll (opinion mining), and CrisisConnect (emergency response), SarkarSahayak evolves from a scheme discovery tool into India's most comprehensive AI-powered civic engagement platform - transforming how 900 million citizens interact with their government.

**From Benefits to Accountability**: Discover schemes → Apply seamlessly → Report issues → Track resolution → Provide feedback → Influence policy - all in one platform, all in your language.

---

**Document Version**: 1.0  
**Last Updated**: February 14, 2026  
**Team**: Chai & Algorithms
