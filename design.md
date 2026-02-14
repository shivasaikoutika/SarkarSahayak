# SarkarSahayak - System Design Document

## Table of Contents
1. [System Overview](#system-overview)
2. [Architecture Design](#architecture-design)
3. [Component Design](#component-design)
4. [Data Design](#data-design)
5. [AI/ML Design](#aiml-design)
6. [User Interface Design](#user-interface-design)
7. [Security Design](#security-design)
8. [Deployment Architecture](#deployment-architecture)
9. [Technology Stack](#technology-stack)
10. [Implementation Plan](#implementation-plan)

---

## System Overview

### High-Level Vision
SarkarSahayak is a conversational AI assistant that democratizes access to government welfare schemes across India. It acts as a digital bridge between citizens and government benefits, using natural language processing to understand user needs and recommend relevant schemes in their preferred language.

### Key Design Principles
1. **Simplicity First**: No learning curve, conversational interface
2. **Language No Barrier**: Multilingual from ground up
3. **Privacy by Design**: Minimal data collection, maximum user control
4. **Offline-Ready**: Progressive Web App with offline capabilities
5. **Mobile-First**: Optimized for low-end smartphones and slow networks
6. **Accessibility**: Usable by persons with disabilities and low literacy

### System Goals
- **User Goal**: Find schemes, understand eligibility, get application guidance in < 5 minutes
- **System Goal**: Handle 10,000+ daily users with 95%+ accuracy
- **Business Goal**: Increase scheme awareness and applications by 300%

---

## Architecture Design

### 1. High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        USER LAYER                            │
│  (Web Browser, WhatsApp, Progressive Web App, SMS)          │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                   PRESENTATION LAYER                         │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   Web UI     │  │   WhatsApp   │  │   Voice      │      │
│  │  (Streamlit) │  │   Business   │  │  Interface   │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                    API GATEWAY LAYER                         │
│              (FastAPI / Flask REST API)                      │
│  • Authentication  • Rate Limiting  • Request Routing       │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  APPLICATION LAYER                           │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │  Conversation  │  │     Scheme     │  │  Language    │  │
│  │    Manager     │  │  Recommendation│  │  Processor   │  │
│  └────────────────┘  └────────────────┘  └──────────────┘  │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │   Eligibility  │  │   Analytics    │  │   Notif.     │  │
│  │     Engine     │  │    Service     │  │   Service    │  │
│  └────────────────┘  └────────────────┘  └──────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                      AI/ML LAYER                             │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │  Google Gemini │  │   Translation  │  │    Intent    │  │
│  │   Pro API      │  │      API       │  │  Classifier  │  │
│  └────────────────┘  └────────────────┘  └──────────────┘  │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                      DATA LAYER                              │
│  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐  │
│  │    Schemes     │  │   User Prefs   │  │  Conversation│  │
│  │   Database     │  │   (Optional)   │  │    Cache     │  │
│  │   (MongoDB)    │  │   (Redis)      │  │   (Redis)    │  │
│  └────────────────┘  └────────────────┘  └──────────────┘  │
└─────────────────────────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│                  EXTERNAL SERVICES                           │
│  • Government APIs  • WhatsApp API  • SMS Gateway           │
│  • Google Translate  • Analytics  • CDN                     │
└─────────────────────────────────────────────────────────────┘
```

### 2. Architecture Patterns

#### Microservices Architecture (Future-Ready)
While MVP is monolithic for speed, design is modular for easy service separation:
- **Conversation Service**: Handles chat logic
- **Recommendation Service**: Scheme matching algorithm
- **Translation Service**: Multilingual support
- **Notification Service**: WhatsApp/SMS alerts

#### Event-Driven Design
- User interactions trigger events
- Async processing for analytics, notifications
- Message queue (RabbitMQ/Redis) for scalability

#### Caching Strategy
```
User Request → Check Cache → Cache Hit? Return
                            → Cache Miss → Process → Update Cache → Return
```
- Redis for session data (30 min TTL)
- CDN for static assets
- In-memory cache for scheme data (1 hour TTL)

---

## Component Design

### 1. Conversation Manager

**Purpose**: Orchestrates the chat flow, maintains context, generates responses

**Responsibilities**:
- Parse user input
- Maintain conversation state (current question, collected info)
- Decide next question to ask
- Generate natural language responses
- Handle edge cases (unclear input, off-topic queries)

**Algorithm - Conversation Flow**:
```python
class ConversationManager:
    def __init__(self):
        self.state = {
            "stage": "greeting",  # greeting → profiling → recommendation → application
            "user_data": {},
            "schemes_found": [],
            "current_question": None
        }
    
    def process_message(self, user_message):
        # 1. Detect intent
        intent = self.classify_intent(user_message)
        
        # 2. Extract entities
        entities = self.extract_entities(user_message)
        
        # 3. Update user profile
        self.update_profile(entities)
        
        # 4. Determine next action
        if self.state["stage"] == "greeting":
            return self.start_profiling()
        
        elif self.state["stage"] == "profiling":
            if self.has_enough_data():
                schemes = self.find_schemes()
                self.state["stage"] = "recommendation"
                return self.present_schemes(schemes)
            else:
                return self.ask_next_question()
        
        elif self.state["stage"] == "recommendation":
            return self.handle_scheme_query(user_message)
        
        elif self.state["stage"] == "application":
            return self.guide_application(user_message)
    
    def has_enough_data(self):
        required = ["age", "state", "occupation"]
        return all(field in self.state["user_data"] for field in required)
```

**State Management**:
- Session-based (cookie/localStorage)
- Conversation history stored in Redis (TTL: 24 hours)
- Ability to resume conversation

---

### 2. Scheme Recommendation Engine

**Purpose**: Match user profile to eligible schemes with confidence scores

**Core Algorithm**:
```python
class SchemeRecommender:
    def recommend(self, user_profile):
        """
        Returns list of schemes ranked by relevance
        """
        schemes = self.load_all_schemes()
        scored_schemes = []
        
        for scheme in schemes:
            # 1. Hard filters (must match)
            if not self.meets_hard_criteria(user_profile, scheme):
                continue
            
            # 2. Calculate match score (0-100)
            score = self.calculate_match_score(user_profile, scheme)
            
            # 3. Add explanation
            explanation = self.generate_explanation(user_profile, scheme)
            
            scored_schemes.append({
                "scheme": scheme,
                "score": score,
                "explanation": explanation,
                "confidence": self.get_confidence(score)
            })
        
        # 3. Sort by score (descending)
        scored_schemes.sort(key=lambda x: x["score"], reverse=True)
        
        return scored_schemes[:10]  # Top 10
    
    def calculate_match_score(self, user_profile, scheme):
        score = 0
        weights = {
            "category_match": 30,
            "income_benefit_ratio": 25,
            "benefit_amount": 20,
            "ease_of_application": 15,
            "popularity": 10
        }
        
        # Category match
        if user_profile.get("occupation") in scheme.get("target_categories", []):
            score += weights["category_match"]
        
        # Income to benefit ratio
        if scheme.get("benefit_amount"):
            benefit_ratio = scheme["benefit_amount"] / max(user_profile.get("income", 1), 1)
            score += weights["income_benefit_ratio"] * min(benefit_ratio, 1)
        
        # Benefit amount (higher is better)
        score += weights["benefit_amount"] * (min(scheme.get("benefit_amount", 0), 50000) / 50000)
        
        # Ease of application (online > offline)
        if scheme.get("online_application"):
            score += weights["ease_of_application"]
        
        # Popularity (more applications = proven scheme)
        popularity_score = min(scheme.get("total_applications", 0), 1000000) / 1000000
        score += weights["popularity"] * popularity_score
        
        return score
```

**Eligibility Checker**:
```python
def meets_hard_criteria(user_profile, scheme):
    """
    Check if user meets mandatory eligibility
    """
    criteria = scheme.get("eligibility", {})
    
    # Age check
    if "min_age" in criteria:
        if user_profile.get("age", 0) < criteria["min_age"]:
            return False
    if "max_age" in criteria:
        if user_profile.get("age", 999) > criteria["max_age"]:
            return False
    
    # Income check
    if "max_income" in criteria:
        if user_profile.get("income", 0) > criteria["max_income"]:
            return False
    
    # State check
    if "applicable_states" in criteria:
        if user_profile.get("state") not in criteria["applicable_states"]:
            return False
    
    # Custom conditions
    if "conditions" in criteria:
        for condition in criteria["conditions"]:
            if not eval_condition(condition, user_profile):
                return False
    
    return True
```

---

### 3. Language Processor

**Purpose**: Handle multilingual input/output seamlessly

**Architecture**:
```
User Input (Hindi) 
    ↓
Language Detection (Auto-detect or user preference)
    ↓
Translation to English (if needed)
    ↓
Processing (Intent, Entities, Response)
    ↓
Translation to User's Language
    ↓
Response Output (Hindi)
```

**Implementation**:
```python
class LanguageProcessor:
    SUPPORTED_LANGUAGES = {
        "en": "English",
        "hi": "Hindi",
        "ta": "Tamil",
        "bn": "Bengali",
        "mr": "Marathi",
        "te": "Telugu",
        "gu": "Gujarati"
    }
    
    def __init__(self):
        self.translator = GoogleTranslator()
        self.detector = LanguageDetector()
    
    def process_input(self, text, user_lang_pref=None):
        # 1. Detect language
        detected_lang = user_lang_pref or self.detector.detect(text)
        
        # 2. Translate to English for processing
        if detected_lang != "en":
            english_text = self.translator.translate(text, dest="en", src=detected_lang)
        else:
            english_text = text
        
        return english_text, detected_lang
    
    def process_output(self, english_response, target_lang):
        # Translate response back to user's language
        if target_lang != "en":
            translated = self.translator.translate(english_response, dest=target_lang, src="en")
            return translated
        return english_response
    
    def translate_scheme_data(self, scheme, target_lang):
        """
        Translate scheme name and description
        Keep technical terms in original language
        """
        translatable_fields = ["name", "description", "benefits", "documents"]
        
        translated_scheme = scheme.copy()
        for field in translatable_fields:
            if field in scheme:
                translated_scheme[field] = self.translator.translate(
                    scheme[field], 
                    dest=target_lang,
                    src="en"
                )
        
        return translated_scheme
```

---

### 4. User Interface Design

#### Web Interface (Primary)

**Technology**: Streamlit (MVP), React (Production)

**Layout**:
```
┌─────────────────────────────────────────────────┐
│  🇮🇳 SarkarSahayak        🌐 Language ▼  ℹ️ Help │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │  Chat Messages Area                     │   │
│  │                                         │   │
│  │  Bot: नमस्ते! मैं आपकी सरकारी योजनाओं   │   │
│  │       में मदद करूंगा।                  │   │
│  │                                         │   │
│  │  User: मुझे किसान योजना चाहिए          │   │
│  │                                         │   │
│  │  Bot: बढ़िया! कुछ सवाल पूछने की         │   │
│  │       अनुमति दें...                      │   │
│  │                                         │   │
│  │  • आपकी उम्र क्या है?                   │   │
│  │  • आपके पास कितनी जमीन है?              │   │
│  │  • आपका राज्य कौन सा है?                │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │  Type your message...         🎤 Send ➤ │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  Quick Actions:                                 │
│  [👨‍🌾 Farmer] [👶 Parents] [👴 Senior] [💼 Business] │
└─────────────────────────────────────────────────┘
```

**Color Scheme**:
- Primary: Saffron (#FF9933) - Government India theme
- Secondary: Green (#138808) - Eligibility indicator
- Accent: Navy Blue (#000080) - Trust, authority
- Background: White/Light Gray
- Error: Red (#FF0000)

**Responsive Breakpoints**:
- Mobile: 320px - 768px
- Tablet: 769px - 1024px
- Desktop: 1025px+

---

#### Scheme Results Display

```
┌─────────────────────────────────────────────────┐
│  ✅ आपके लिए 8 योजनाएं मिलीं!                    │
├─────────────────────────────────────────────────┤
│                                                 │
│  📋 PM-KISAN (प्रधानमंत्री किसान सम्मान निधि)   │
│  ────────────────────────────────────────       │
│  💰 लाभ: ₹6,000/वर्ष (₹2000 x 3 किस्त)         │
│  ✓  आप योग्य हैं! (95% मैच)                     │
│  📄 जरूरी: आधार, जमीन के कागजात, बैंक          │
│  🔗 [अभी अप्लाई करें] [और जानें]                │
│                                                 │
│  📋 Ayushman Bharat (आयुष्मान भारत)             │
│  ────────────────────────────────────────       │
│  💰 लाभ: ₹5 लाख स्वास्थ्य बीमा                 │
│  ✓  आप योग्य हैं! (88% मैच)                     │
│  📄 जरूरी: राशन कार्ड, आधार                    │
│  🔗 [अभी अप्लाई करें] [और जानें]                │
│                                                 │
│  [अगली 6 योजनाएं देखें ▼]                       │
└─────────────────────────────────────────────────┘
```

---

### 5. WhatsApp Integration Design

**Flow**:
```
User sends message to WhatsApp Business Number
    ↓
WhatsApp Cloud API receives message
    ↓
Webhook sends to our backend
    ↓
Process message through Conversation Manager
    ↓
Generate response
    ↓
Send via WhatsApp API
    ↓
User receives message
```

**Template Messages** (Pre-approved by WhatsApp):
```
Welcome Message:
"नमस्ते! 🇮🇳 मैं सरकारसहायक हूं। मैं आपको सरकारी योजनाओं 
में मदद करूंगा। शुरू करने के लिए 'हाँ' भेजें।"

Scheme Found:
"✅ बढ़िया खबर! आप {{scheme_count}} योजनाओं के 
योग्य हैं। विवरण देखने के लिए लिंक: {{link}}"
```

---

## Data Design

### 1. Scheme Database Schema (MongoDB)

```json
{
  "_id": "pm_kisan_2024",
  "name": {
    "en": "PM-KISAN",
    "hi": "प्रधानमंत्री किसान सम्मान निधि",
    "ta": "பிரதமர் கிசான் சம்மன் நிதி"
  },
  "category": "agriculture",
  "government_level": "central",
  "ministry": "Ministry of Agriculture",
  "launch_date": "2019-02-01",
  "status": "active",
  
  "eligibility": {
    "occupation": ["farmer", "agriculturist"],
    "land_ownership": "required",
    "min_age": 18,
    "income_limits": {
      "max_annual_income": null
    },
    "applicable_states": ["all"],
    "exclusions": [
      "government_employees",
      "income_tax_payers",
      "professionals"
    ],
    "custom_conditions": [
      {
        "field": "land_size",
        "operator": ">",
        "value": 0
      }
    ]
  },
  
  "benefits": {
    "type": "cash_transfer",
    "amount": 6000,
    "currency": "INR",
    "frequency": "yearly",
    "installments": 3,
    "installment_amount": 2000,
    "payment_mode": "DBT",
    "description": {
      "en": "₹6000 per year in 3 installments of ₹2000 directly to bank",
      "hi": "प्रति वर्ष ₹6000, 3 किस्तों में ₹2000 सीधे बैंक में"
    }
  },
  
  "application": {
    "online": true,
    "offline": true,
    "portal_url": "https://pmkisan.gov.in/",
    "helpline": "155261",
    "documents_required": [
      {
        "name": "Aadhaar Card",
        "mandatory": true
      },
      {
        "name": "Land Ownership Papers",
        "mandatory": true
      },
      {
        "name": "Bank Account Details",
        "mandatory": true
      }
    ],
    "application_deadline": null,
    "processing_time_days": 30
  },
  
  "metadata": {
    "total_beneficiaries": 110000000,
    "total_disbursed": 2500000000000,
    "last_updated": "2026-02-01T00:00:00Z",
    "source_url": "https://pmkisan.gov.in/",
    "verified": true,
    "popularity_score": 95
  },
  
  "tags": ["farmer", "agriculture", "cash", "DBT", "popular"],
  "search_keywords": ["kisan", "farmer", "agriculture", "farming", "किसान"]
}
```

### 2. User Session Schema (Redis)

```json
{
  "session_id": "uuid-1234-5678",
  "created_at": "2026-02-14T10:30:00Z",
  "last_active": "2026-02-14T10:35:00Z",
  "ttl": 1800,
  
  "user_profile": {
    "age": 45,
    "gender": "male",
    "state": "Punjab",
    "district": "Ludhiana",
    "occupation": "farmer",
    "annual_income": 250000,
    "family_size": 5,
    "has_daughters": true,
    "land_size_acres": 3
  },
  
  "conversation_state": {
    "stage": "recommendation",
    "language": "hi",
    "questions_asked": ["age", "occupation", "state", "income"],
    "schemes_shown": ["pm_kisan_2024", "ayushman_bharat_2024"],
    "user_intent": "find_agriculture_schemes"
  },
  
  "conversation_history": [
    {
      "timestamp": "2026-02-14T10:30:15Z",
      "role": "user",
      "message": "मुझे किसान योजना चाहिए",
      "language": "hi"
    },
    {
      "timestamp": "2026-02-14T10:30:20Z",
      "role": "bot",
      "message": "बिल्कुल! मैं आपकी मदद करूंगा। आपकी उम्र क्या है?",
      "language": "hi"
    }
  ],
  
  "preferences": {
    "language": "hi",
    "voice_enabled": false,
    "notifications": true
  }
}
```

### 3. Analytics Events Schema

```json
{
  "event_id": "uuid",
  "timestamp": "2026-02-14T10:30:00Z",
  "event_type": "scheme_recommended",
  
  "user_data": {
    "session_id": "uuid-1234",
    "state": "Punjab",
    "language": "hi",
    "user_segment": "farmer"
  },
  
  "event_data": {
    "scheme_id": "pm_kisan_2024",
    "recommendation_score": 95,
    "rank": 1,
    "user_action": "clicked_apply"
  },
  
  "metadata": {
    "platform": "web",
    "device_type": "mobile",
    "response_time_ms": 1200
  }
}
```

---

## AI/ML Design

### 1. Intent Classification

**Model**: Google Gemini Pro (via API)

**Intents**:
- `greeting` - User says hi, hello
- `scheme_query` - Asking about schemes
- `eligibility_check` - "Am I eligible?"
- `application_help` - "How to apply?"
- `document_query` - "What documents needed?"
- `complaint` - Issues with application
- `off_topic` - Unrelated queries

**Prompt Engineering**:
```python
INTENT_PROMPT = """
You are an intent classifier for a government schemes chatbot.
Classify the user's message into one of these intents:
- greeting
- scheme_query
- eligibility_check  
- application_help
- document_query
- complaint
- off_topic

User message: "{user_message}"
Language: {language}

Return ONLY the intent name, nothing else.
"""
```

### 2. Entity Extraction

**Entities to Extract**:
- `age`: Number
- `occupation`: Farmer, student, senior_citizen, business_owner, etc.
- `income`: Annual income in rupees
- `state`: Indian state name
- `category`: Agriculture, health, education, etc.

**Gemini Prompt**:
```python
ENTITY_EXTRACTION_PROMPT = """
Extract structured information from the user's message.

User: "{user_message}"

Extract these entities if present:
- age (number)
- occupation (farmer/student/business/senior/other)
- income (annual in rupees)
- state (Indian state)
- has_daughters (yes/no)
- land_size (acres)

Return as JSON. If entity not found, use null.
Example: {{"age": 45, "occupation": "farmer", "income": 250000}}

JSON:
"""
```

### 3. Response Generation

**Approach**: Hybrid (Template + AI)

**For Structured Responses** (eligibility, scheme details):
- Use templates for consistency
- Faster, cheaper, predictable

**For Open-Ended Queries**:
- Use Gemini for natural responses
- Context-aware, empathetic

```python
def generate_response(user_message, context):
    if context["stage"] == "greeting":
        return TEMPLATE_GREETING[context["language"]]
    
    elif context["stage"] == "profiling":
        next_question = get_next_question(context["collected_data"])
        return TEMPLATE_QUESTIONS[next_question][context["language"]]
    
    elif context["stage"] == "recommendation":
        # Use template for scheme display
        return format_schemes_response(context["schemes_found"], context["language"])
    
    else:
        # Use Gemini for complex queries
        return ask_gemini(user_message, context)
```

### 4. Confidence Scoring

**Algorithm**:
```python
def calculate_confidence(match_score, data_quality):
    """
    Confidence = (Match Score * 0.7) + (Data Quality * 0.3)
    
    Match Score: How well user matches eligibility (0-100)
    Data Quality: How complete is user profile (0-100)
    """
    
    confidence = (match_score * 0.7) + (data_quality * 0.3)
    
    if confidence >= 85:
        return "high", "आप निश्चित रूप से योग्य हैं"
    elif confidence >= 70:
        return "medium", "आप शायद योग्य हैं"
    else:
        return "low", "कृपया और जानकारी दें"
```

---

## Security Design

### 1. Data Privacy

**Principles**:
- **Minimal Collection**: Only ask what's necessary
- **No PII Storage**: Don't store Aadhaar, bank details
- **Encrypted Transit**: HTTPS only
- **Encrypted at Rest**: MongoDB encryption
- **Right to Delete**: Users can clear conversation

**Implementation**:
```python
# Don't store sensitive data
BLOCKED_FIELDS = [
    "aadhaar_number",
    "pan_card",
    "bank_account",
    "phone_number",
    "email"  # Unless user explicitly opts in
]

def sanitize_user_data(data):
    for field in BLOCKED_FIELDS:
        if field in data:
            del data[field]
    return data
```

### 2. API Security

**Rate Limiting**:
```python
# Per IP: 100 requests/minute
# Per Session: 30 requests/minute
# Per API key: 10,000 requests/day

@app.route('/api/chat')
@rate_limit(requests=30, window=60)  # 30 req/min
def chat_endpoint():
    pass
```

**Authentication** (for admin dashboard):
- JWT tokens
- Role-based access control (RBAC)
- 2FA for admin login

**Input Validation**:
```python
def validate_user_input(message):
    # 1. Length check
    if len(message) > 500:
        raise ValueError("Message too long")
    
    # 2. Injection prevention
    if contains_sql_injection(message):
        raise SecurityError("Invalid input")
    
    # 3. XSS prevention
    message = sanitize_html(message)
    
    return message
```

### 3. Infrastructure Security

- **Firewall**: Only ports 80, 443 open
- **DDoS Protection**: Cloudflare
- **Secrets Management**: AWS Secrets Manager / Environment variables
- **Logging**: All API calls logged (without PII)
- **Monitoring**: Alert on suspicious patterns

---

## Deployment Architecture

### Phase 1: MVP (Hackathon)

```
┌──────────────────────────────────────────┐
│         Streamlit Cloud (Free)           │
│  • Frontend + Backend in one             │
│  • Auto-deploy from GitHub               │
│  • HTTPS enabled                         │
└──────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────┐
│      Google Gemini API (Free Tier)       │
│  • 1500 requests/day                     │
└──────────────────────────────────────────┘
          │
          ▼
┌──────────────────────────────────────────┐
│    MongoDB Atlas (Free M0 Cluster)       │
│  • 512 MB storage                        │
│  • Shared cluster                        │
└──────────────────────────────────────────┘
```

**Estimated Cost**: ₹0 (All free tiers)

---

### Phase 2: Production (Post-Hackathon)

```
┌────────────────────────────────────────────────────┐
│               Cloudflare (CDN + DDoS)              │
└────────────────────┬───────────────────────────────┘
                     │
                     ▼
┌────────────────────────────────────────────────────┐
│            Load Balancer (AWS ALB)                 │
└───────┬──────────────────────────┬─────────────────┘
        │                          │
        ▼                          ▼
┌──────────────┐          ┌──────────────┐
│  Web Server  │          │  Web Server  │
│  (EC2 x2)    │          │  (EC2 x2)    │
│  FastAPI     │          │  FastAPI     │
└──────┬───────┘          └──────┬───────┘
       │                         │
       └────────┬────────────────┘
                ▼
       ┌─────────────────┐
       │  Redis Cluster  │
       │  (ElastiCache)  │
       └─────────────────┘
                │
                ▼
       ┌─────────────────┐
       │  MongoDB Atlas  │
       │  (M10 cluster)  │
       └─────────────────┘
```

**Components**:
- **Frontend**: React (Vercel/Netlify)
- **Backend**: FastAPI on AWS EC2 (2 instances)
- **Cache**: Redis (AWS ElastiCache)
- **Database**: MongoDB Atlas (M10)
- **CDN**: Cloudflare
- **Monitoring**: Datadog / AWS CloudWatch
- **CI/CD**: GitHub Actions

**Estimated Cost**: ₹15,000-20,000/month

---

## Technology Stack

### Frontend
- **Framework**: Streamlit (MVP), React (Production)
- **UI Library**: Tailwind CSS, shadcn/ui
- **Icons**: Lucide React
- **State**: React Context / Redux
- **PWA**: Workbox for offline support

### Backend
- **Framework**: Flask (MVP), FastAPI (Production)
- **Language**: Python 3.11+
- **API Docs**: Swagger / OpenAPI

### AI/ML
- **LLM**: Google Gemini Pro API
- **Translation**: Google Translate API
- **NLP**: spaCy (for entity extraction fallback)
- **Speech**: Google Cloud Speech-to-Text (future)

### Database
- **Primary**: MongoDB Atlas
- **Cache**: Redis
- **Search**: MongoDB Atlas Search / Elasticsearch (future)

### DevOps
- **Version Control**: GitHub
- **CI/CD**: GitHub Actions
- **Hosting**: Streamlit Cloud (MVP), AWS EC2 (Prod)
- **Monitoring**: Sentry (errors), Mixpanel (analytics)
- **Logging**: Python logging + CloudWatch

### External APIs
- **WhatsApp**: WhatsApp Business Cloud API
- **SMS**: Twilio / MSG91
- **Payment**: Not required (free service)

---

## Implementation Plan

### Week 1: Foundation
- [ ] Set up GitHub repo
- [ ] Configure MongoDB Atlas
- [ ] Get Gemini API key
- [ ] Create basic Streamlit UI
- [ ] Implement conversation flow (hardcoded)

### Week 2: Core Features
- [ ] Build scheme database (30 schemes)
- [ ] Implement eligibility engine
- [ ] Integrate Gemini API
- [ ] Add Hindi translation
- [ ] Create recommendation algorithm

### Week 3: Polish & Test
- [ ] Add remaining languages (5 more)
- [ ] Implement caching
- [ ] User testing (20 people)
- [ ] Fix bugs
- [ ] Write documentation

### Week 4: Launch Prep
- [ ] Deploy to Streamlit Cloud
- [ ] Create demo video
- [ ] Prepare pitch deck
- [ ] Practice presentation
- [ ] Submit to hackathon

---

## Testing Strategy

### Unit Tests
```python
def test_eligibility_checker():
    user = {"age": 45, "occupation": "farmer", "state": "Punjab"}
    scheme = load_scheme("pm_kisan")
    assert meets_eligibility(user, scheme) == True

def test_recommendation_scoring():
    user = {"age": 65, "income": 150000}
    schemes = recommend_schemes(user)
    assert schemes[0]["scheme_id"] == "atal_pension"  # Best match
```

### Integration Tests
- API endpoint testing
- Database connection tests
- External API mocking

### User Acceptance Testing
- 20 users from target demographic
- Tasks: Find scheme, check eligibility, understand benefits
- Success: 80% complete tasks without help

### Performance Testing
- Load test: 100 concurrent users
- Response time: < 3 seconds
- Error rate: < 1%

---

## Monitoring & Analytics

### Key Metrics
- **User Engagement**: DAU, session duration, messages per session
- **Conversion**: % users who click "Apply Now"
- **Performance**: API response time, error rate
- **AI Quality**: User satisfaction (thumbs up/down)

### Dashboards
- Real-time usage map (state-wise)
- Most searched schemes
- Conversation drop-off points
- Language distribution

---

## Future Platform Architecture (Months 7-12)

### Integrated Civic Engagement Platform

Building on SarkarSahayak's foundation, we'll expand into a comprehensive platform integrating:
- **NagrikVaani**: AI Complaint Management
- **JantaKaPoll**: Public Opinion Mining
- **CrisisConnect**: Emergency Response

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                        UNIFIED USER LAYER                                    │
│         Web App | Mobile PWA | WhatsApp Bot | SMS Gateway                   │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                       UNIFIED API GATEWAY                                    │
│              Authentication | Rate Limiting | Request Routing               │
└────────────────────────────┬────────────────────────────────────────────────┘
                             │
        ┌────────────────────┼────────────────────┬───────────────────┐
        ▼                    ▼                    ▼                   ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐  ┌──────────────┐
│  SarkarSahayak  │  │  NagrikVaani    │  │  JantaKaPoll    │  │ CrisisConnect│
│    (Schemes)    │  │  (Complaints)   │  │  (Opinions)     │  │ (Emergency)  │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘  └──────┬───────┘
         │                    │                     │                  │
         └────────────────────┴─────────────────────┴──────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         SHARED AI/ML LAYER                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │ Google Gemini│  │  Translation │  │   Sentiment  │  │    Vision    │   │
│  │     Pro      │  │     API      │  │   Analysis   │  │     API      │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                     │
│  │    Intent    │  │  Clustering  │  │   Topic      │                     │
│  │ Classifier   │  │  (Duplicates)│  │  Modeling    │                     │
│  └──────────────┘  └──────────────┘  └──────────────┘                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         UNIFIED DATA LAYER                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │
│  │   Schemes    │  │  Complaints  │  │   Sentiment  │  │   Emergency  │   │
│  │   Database   │  │   Database   │  │     Data     │  │     Alerts   │   │
│  │  (MongoDB)   │  │  (MongoDB)   │  │  (MongoDB)   │  │   (Redis)    │   │
│  └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘   │
│  ┌──────────────────────────────────────────────────────────────────────┐  │
│  │            Shared Cache Layer (Redis) + Search (Elasticsearch)       │  │
│  └──────────────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────────────┘
                                        │
                                        ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        EXTERNAL INTEGRATIONS                                 │
│  • Government APIs  • Social Media APIs  • Weather APIs  • Disaster DB     │
│  • WhatsApp API  • SMS Gateway  • News Sources  • Google Maps             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Module Interactions & Data Flow

**Cross-Module Synergies**:

1. **Scheme Discovery → Complaint Filing**:
   ```
   User discovers PM-KISAN through SarkarSahayak
       ↓
   Applies for scheme
       ↓
   Payment delayed? → Switch to NagrikVaani module
       ↓
   File complaint: "PM-KISAN payment not received"
       ↓
   Auto-categorized, sent to Agriculture Department
       ↓
   User notified when resolved
   ```

2. **User Feedback → Opinion Mining**:
   ```
   5000 users search for "unemployment scheme" in SarkarSahayak
       ↓
   JantaKaPoll detects spike in unemployment-related queries
       ↓
   Analyzes sentiment: 70% users frustrated, demand more job schemes
       ↓
   Dashboard alerts policymakers: "Unemployment schemes trending"
       ↓
   Government can launch targeted programs
   ```

3. **Emergency → Scheme Access**:
   ```
   Cyclone warning in Odisha
       ↓
   CrisisConnect sends alerts to 500K users
       ↓
   Shows evacuation centers + emergency contacts
       ↓
   Post-cyclone: Auto-recommend disaster relief schemes
       ↓
   "Your house damaged? Eligible for PM Awas reconstruction"
       ↓
   Fast-track scheme application for disaster victims
   ```

4. **Complaint Patterns → Scheme Improvements**:
   ```
   NagrikVaani receives 1000 complaints: "Aadhaar linking issue in Ayushman Bharat"
       ↓
   AI clusters similar complaints
       ↓
   JantaKaPoll confirms negative sentiment on social media
       ↓
   Alert sent to Health Ministry: "Major implementation issue detected"
       ↓
   Government fixes process
       ↓
   SarkarSahayak updates scheme info with clearer instructions
   ```

---

## Extended Component Designs

### NagrikVaani - Complaint Management Engine

**Core Components**:

1. **Complaint Intake Service**
   ```python
   class ComplaintIntakeService:
       def process_complaint(complaint_text, images, location):
           # 1. Language detection
           language = detect_language(complaint_text)
           
           # 2. Translate to English for processing
           english_text = translate(complaint_text, target="en")
           
           # 3. Extract entities
           entities = extract_entities(english_text)
           # Output: {category, location, severity, department}
           
           # 4. Classify priority
           priority = classify_priority(english_text, entities)
           # CRITICAL | HIGH | MEDIUM | LOW
           
           # 5. Sentiment analysis
           sentiment = analyze_sentiment(english_text)
           # angry/frustrated/neutral/satisfied
           
           # 6. Image verification (if attached)
           if images:
               authenticity = verify_image_authenticity(images)
               context = extract_image_context(images)
           
           return {
               "complaint_id": generate_id(),
               "category": entities.category,
               "department": map_to_department(entities),
               "priority": priority,
               "sentiment": sentiment,
               "verified": authenticity
           }
   ```

2. **Duplicate Detection Engine**
   ```python
   class DuplicateDetector:
       def find_similar_complaints(new_complaint):
           # 1. Generate embedding
           embedding = generate_text_embedding(new_complaint.text)
           
           # 2. Search similar in vector DB
           similar = vector_db.search(embedding, threshold=0.85)
           
           # 3. Additional filters
           similar = filter_by_location(similar, radius_km=5)
           similar = filter_by_timeframe(similar, days=30)
           
           # 4. Cluster if threshold met
           if len(similar) > 10:
               create_cluster({
                   "cluster_id": "pothole_zone_A",
                   "complaints": similar + [new_complaint],
                   "unified_description": summarize_complaints(similar),
                   "affected_citizens": len(similar),
                   "priority": "HIGH"  # escalate clustered issues
               })
           
           return similar
   ```

3. **Auto-Assignment System**
   ```python
   DEPARTMENT_MAPPING = {
       "potholes": "PWD",
       "water_supply": "Water Department",
       "garbage": "Sanitation Department",
       "electricity": "Power Distribution",
       "healthcare": "Health Department",
       "education": "Education Department"
   }
   
   def assign_complaint(complaint):
       department = DEPARTMENT_MAPPING[complaint.category]
       
       # Get responsible officer based on location
       officer = get_ward_officer(complaint.location, department)
       
       # Create assignment
       assignment = {
           "complaint_id": complaint.id,
           "assigned_to": officer,
           "assigned_at": now(),
           "expected_resolution": predict_resolution_time(complaint),
           "escalation_threshold": 7  # days
       }
       
       # Send notifications
       notify_officer(officer, complaint)
       notify_citizen(complaint.citizen_id, assignment)
       
       return assignment
   ```

---

### JantaKaPoll - Opinion Mining Engine

**Core Components**:

1. **Social Media Scraper**
   ```python
   class SocialMediaMonitor:
       def collect_opinions(scheme_name, timeframe="7d"):
           opinions = []
           
           # Twitter
           tweets = twitter_api.search(
               query=f"{scheme_name} OR #{scheme_hashtag}",
               lang=["hi", "en", "ta", "bn"],
               count=10000
           )
           opinions.extend(tweets)
           
           # Reddit
           reddit_posts = reddit.search(
               subreddit="india",
               query=scheme_name,
               time_filter="week"
           )
           opinions.extend(reddit_posts)
           
           # News Comments
           news_comments = scrape_news_comments(scheme_name)
           opinions.extend(news_comments)
           
           # YouTube
           youtube_comments = get_youtube_comments(
               search_term=f"{scheme_name} government"
           )
           opinions.extend(youtube_comments)
           
           return opinions
   ```

2. **Sentiment Analyzer**
   ```python
   class SentimentAnalyzer:
       def analyze_scheme_sentiment(opinions, scheme_name):
           results = {
               "scheme": scheme_name,
               "total_mentions": len(opinions),
               "sentiment_breakdown": {
                   "positive": 0,
                   "negative": 0,
                   "neutral": 0
               },
               "top_topics": [],
               "regional_sentiment": {},
               "trending_concerns": []
           }
           
           for opinion in opinions:
               # Translate to English
               text = translate_if_needed(opinion.text)
               
               # Classify sentiment
               sentiment = classify_sentiment(text)
               results["sentiment_breakdown"][sentiment] += 1
               
               # Extract topics
               topics = extract_topics(text)
               results["top_topics"].extend(topics)
               
               # Regional analysis
               region = detect_region(opinion)
               if region not in results["regional_sentiment"]:
                   results["regional_sentiment"][region] = {
                       "positive": 0, "negative": 0, "neutral": 0
                   }
               results["regional_sentiment"][region][sentiment] += 1
           
           # Identify trending concerns
           results["trending_concerns"] = find_common_complaints(
               [o for o in opinions if o.sentiment == "negative"]
           )
           
           return results
   ```

3. **Trend Detection**
   ```python
   class TrendDetector:
       def detect_sentiment_shifts(scheme_name, window_days=30):
           # Get historical sentiment data
           historical = db.get_sentiment_history(scheme_name, days=window_days)
           
           # Calculate daily sentiment scores
           daily_scores = calculate_daily_scores(historical)
           
           # Detect significant changes
           shifts = []
           for i in range(1, len(daily_scores)):
               change = daily_scores[i] - daily_scores[i-1]
               
               if abs(change) > 15:  # 15% shift threshold
                   shifts.append({
                       "date": daily_scores[i].date,
                       "change": change,
                       "direction": "positive" if change > 0 else "negative",
                       "likely_cause": identify_cause(scheme_name, daily_scores[i].date)
                   })
           
           # Alert if negative trend
           if any(s["direction"] == "negative" and s["change"] < -20 for s in shifts):
               send_alert_to_government({
                   "scheme": scheme_name,
                   "alert": "MAJOR NEGATIVE SENTIMENT SHIFT",
                   "details": shifts
               })
           
           return shifts
   ```

---

### CrisisConnect - Emergency Response System

**Core Components**:

1. **Disaster Detection & Alert System**
   ```python
   class DisasterMonitor:
       def monitor_disasters():
           # Monitor multiple sources
           weather_alerts = check_imd_alerts()  # India Meteorological Dept
           earthquake_data = check_seismic_activity()
           flood_warnings = check_cwc_data()  # Central Water Commission
           
           active_disasters = []
           
           for alert in weather_alerts:
               if alert.severity >= "SEVERE":
                   disaster = {
                       "type": alert.type,  # cyclone/flood/heatwave
                       "affected_regions": alert.regions,
                       "severity": alert.severity,
                       "estimated_affected": get_population(alert.regions),
                       "timeline": alert.timeline
                   }
                   active_disasters.append(disaster)
                   
                   # Trigger emergency mode
                   activate_emergency_mode(disaster)
           
           return active_disasters
   
       def activate_emergency_mode(disaster):
           # Get affected users
           affected_users = db.get_users_in_region(disaster.affected_regions)
           
           # Send multi-channel alerts
           for user in affected_users:
               # WhatsApp (primary)
               send_whatsapp_alert(user, disaster)
               
               # SMS (backup)
               send_sms_alert(user, disaster)
               
               # In-app notification
               send_push_notification(user, disaster)
           
           # Switch UI to emergency mode
           enable_emergency_ui(disaster.affected_regions)
           
           # Show relevant schemes
           relief_schemes = get_disaster_schemes(disaster.type)
           make_schemes_prominent(relief_schemes)
   ```

2. **SOS Handler**
   ```python
   class SOSHandler:
       def process_sos(user_id, message, location, images=None):
           # 1. Extract emergency details
           emergency = {
               "user_id": user_id,
               "type": classify_emergency(message),  # medical/trapped/food/water
               "severity": assess_severity(message),
               "location": location,
               "timestamp": now()
           }
           
           # 2. Verify authenticity
           authenticity_score = verify_sos_authenticity(
               user_history=get_user_history(user_id),
               message=message,
               images=images,
               location_confidence=location.accuracy
           )
           
           if authenticity_score < 0.7:
               flag_for_manual_review(emergency)
               return
           
           # 3. Find nearest rescue team
           rescue_team = find_nearest_team(
               location=location,
               emergency_type=emergency.type
           )
           
           # 4. Dispatch
           dispatch = {
               "sos_id": emergency.id,
               "assigned_team": rescue_team,
               "priority": calculate_priority(emergency),
               "eta": calculate_eta(rescue_team.location, location)
           }
           
           # 5. Notify all parties
           notify_rescue_team(rescue_team, emergency, dispatch)
           notify_user(user_id, f"Help is on the way. ETA: {dispatch.eta} mins")
           
           # 6. Track in real-time
           track_rescue_operation(dispatch)
           
           return dispatch
   ```

3. **Resource Coordinator**
   ```python
   class ResourceCoordinator:
       def coordinate_relief_resources(disaster):
           # 1. Identify needs
           needs = assess_disaster_needs(disaster)
           # Output: {food: 50000, water: 100000, medical: 20, shelter: 500}
           
           # 2. Map available resources
           resources = {
               "relief_camps": get_nearby_camps(disaster.region),
               "hospitals": get_functional_hospitals(disaster.region),
               "food_centers": get_food_distribution_points(disaster.region),
               "evacuation_routes": calculate_safe_routes(disaster)
           }
           
           # 3. Optimize allocation
           allocation = optimize_resource_distribution(needs, resources)
           
           # 4. Update users in real-time
           for user in get_affected_users(disaster.region):
               nearest_resources = find_nearest_resources(user.location, resources)
               
               send_resource_info(user, {
                   "nearest_relief_camp": nearest_resources.camp,
                   "distance": nearest_resources.distance,
                   "availability": check_capacity(nearest_resources.camp),
                   "evacuation_route": get_route(user.location, nearest_resources.camp)
               })
           
           return allocation
   ```

---

## Unified Dashboard for Government Officials

**Multi-Module Analytics Platform**:

```
┌───────────────────────────────────────────────────────────────────┐
│                    SARKARSAHAYAK ADMIN DASHBOARD                   │
├───────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌──────────┐│
│  │  Schemes    │  │ Complaints  │  │  Opinion    │  │Emergency ││
│  │  Module     │  │   Module    │  │   Module    │  │  Module  ││
│  └─────────────┘  └─────────────┘  └─────────────┘  └──────────┘│
│                                                                   │
│  Key Metrics (Today):                                             │
│  • 50,000 scheme searches                                         │
│  • 5,000 new applications                                         │
│  • 1,200 new complaints                                           │
│  • Sentiment: 65% positive, 35% negative                          │
│  • 2 active disaster alerts                                       │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐  │
│  │         INTEGRATED INSIGHTS (AI-Generated)                  │  │
│  ├────────────────────────────────────────────────────────────┤  │
│  │ ⚠️  ALERT: PM-KISAN complaints increased 40% this week     │  │
│  │    → 800 complaints about delayed payments                 │  │
│  │    → Negative sentiment trending on social media (-25%)    │  │
│  │    → Recommendation: Investigate payment processing        │  │
│  │                                                             │  │
│  │ ✅  SUCCESS: Ayushman Bharat sentiment improved (+15%)     │  │
│  │    → Faster card issuance appreciated                      │  │
│  │    → Complaints reduced by 30%                             │  │
│  │                                                             │  │
│  │ 🚨  DISASTER: Cyclone warning in Odisha                    │  │
│  │    → 500K users alerted                                    │  │
│  │    → 50 relief camps activated                             │  │
│  │    → Fast-track disaster relief scheme applications        │  │
│  └────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────┘
```

---

## Success Criteria (Updated)

### Technical (Phase 1 - MVP)
✅ 95%+ uptime  
✅ <3s response time  
✅ Support 1000+ concurrent users  
✅ 7 languages working  
✅ 50+ schemes in database  

### Technical (Phase 3 - Integrated Platform)
✅ 99.9% uptime (mission-critical during disasters)  
✅ <2s response time across all modules  
✅ Support 50,000+ concurrent users  
✅ 15 languages across all modules  
✅ 200+ schemes + 10,000+ complaints/day + 1M social posts analyzed/day  

### User Impact (Phase 1)
✅ 10,000+ unique users in Month 1  
✅ 60%+ complete eligibility check  
✅ 40%+ click "Apply Now"  
✅ 4.5+ user rating  

### User Impact (Phase 3 - Integrated Platform)
✅ 10 Million active users by Year 2  
✅ 80%+ user retention (sticky multi-module platform)  
✅ 50% reduction in complaint resolution time (45 days → 7 days)  
✅ 1000+ lives saved through emergency alerts  
✅ ₹10,000 crores in benefits claimed through platform  
✅ Real-time citizen feedback loop to government  

### Hackathon
✅ Working demo  
✅ Clean codebase  
✅ Impressive presentation  
✅ Clear social impact  

---

**Document Version**: 1.0  
**Last Updated**: February 14, 2026  
**Team**: Chai & Algorithms  
**Next Review**: Post-Hackathon
