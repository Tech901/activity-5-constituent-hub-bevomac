---
title: "Activity 5 Reflection - Constituent Services Hub"
type: reflection
version: "1.0.0"
---

# Activity 5 Reflection

Answer each question in 3-5 sentences. Thoughtful, specific responses earn full credit.

## 1. PII Redaction in Practice

What PII categories did the Azure AI Language service detect in the Memphis 311 complaints? Did you notice any false positives (non-PII flagged as sensitive) or false negatives (PII that was missed)? How might PII detection requirements differ between a government agency processing citizen complaints and a commercial customer service system?

The service detected six categories across the 10 complaints: Person, USSocialSecurityNumber, Address, PhoneNumber, Email, and PersonType — 22 entities total. There were a couple of interesting false positives: “family” and “neighbors” got flagged as PersonType, “now” was tagged as DateTime, and the case number “311-2025-044” was misclassified as a PhoneNumber. A government agency needs to be aggressive about redaction for legal compliance, while a commercial system often needs to retain contact info to actually follow up with the customer.

## 2. Sentiment as a Routing Signal

How could sentiment analysis be used to prioritize Memphis 311 complaints — for example, routing highly negative complaints to senior staff? What are the risks of relying solely on sentiment for prioritization (consider complaints that are neutral in tone but urgent in nature)? How would you combine sentiment with other signals for a more robust routing system?

The pipeline correctly scored the sewer backup complaint (three days, terrible smell) as 99% negative and the pothole/SSN complaint as 99% negative — both good candidates for urgent routing. The risk shows up in Complaint 6 (graffiti at Overton Park) which scored 92% neutral despite being a vandalism report. Combining sentiment with key phrases like “sewer,” “pothole,” and “graffiti” from the same results would create a more reliable urgency signal than tone alone.

## 3. Multilingual Challenges

How accurately did the Language service detect the language of short versus long text samples? What challenges might arise with code-switching (mixing languages in a single message), which is common in multilingual communities? How would you handle a complaint where language detection confidence is low?

Spanish and Vietnamese were both detected at 100% confidence, and the translations were accurate — the Vietnamese noise complaint translated cleanly even after PII redaction. One interesting artifact was that sentiment analysis ran on the Vietnamese text before translation, extracting fragmented key phrases like “Tôi muốn báo cáo” instead of meaningful English topics. For low-confidence detections like Summer Avenue in the Vietnamese complaint (0.67), flagging for human review rather than silently passing it through would be the safer approach.

## 4. CLU vs. Keyword Matching

Compare the CLU model's intent classification with the keyword-based fallback. In what scenarios did CLU perform better, and where did keyword matching suffice? What are the trade-offs of training and maintaining a custom CLU model versus using a simpler rule-based approach for a city's 311 system? *(If CLU was not configured in your environment, discuss how you would expect it to differ based on the training data in `data/intent_examples.json`.)*

CLU wasn’t configured so keyword matching handled everything. It did well on clear complaints — potholes, sewers, streetlights all correctly hit report-issue at 100% confidence. It fell apart on the Spanish complaint (Complaint 3) and Vietnamese complaint (Complaint 5), both returning ask-question at 0% confidence because no English keywords matched the redacted foreign-language text. A trained CLU model would handle those cases correctly since it works on meaning rather than exact keyword matches.

## 5. Pipeline Design

Which step in your NLP pipeline was most critical to get right first, and why? If you were deploying this pipeline for production use handling thousands of complaints daily, what monitoring, fallback mechanisms, or error handling would you add? How would you measure pipeline health over time?

PII redaction had to be right first — every other step processes its output, so a miss there means sensitive data flows into sentiment, translation, and intent logs. One thing worth monitoring in production is the false positive rate on PersonType, since redacting words like “neighbors” and “family” slightly degrades the readability of translated and analyzed text downstream. I’d also add alerting on steps completing at under 100% and track per-category PII hit rates over time to catch model drift.​​​​​​​​​​​​​​​​
