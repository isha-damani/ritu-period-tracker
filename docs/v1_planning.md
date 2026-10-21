# Period Tracker: v1 Planning

**Date:** 21 October 2026

## Platform Plan

Build it as a **website first**, then convert it into an app later, because neither of us is familiar with app development.

## Basic Functionality

- **Calendar**
  - Period dates are marked in one color.
  - Predicted period dates are marked in a different color.
- **Prediction**
  - Predict the next period date and the cycle length.
  - Optional: predict ovulation.
- **Current cycle status**
  - Show the current cycle phase and the days remaining until the next period (or the day of the period, if currently on it).
  - Optional: show ovulation.
- **Tutorials**
  - Not tutorials in the strict sense. They are "all you need to know about the cycle, its phases, symptoms, etc."
  - **Accessible to everyone:** active users, people who haven't had their period yet, and partners.
- **Chatbot (RAG)**
  - It will **not diagnose**.

## Pages Plan

*(Very surface level.)*

### 1. Signup

The signup flow starts with a few questions:

1. **Have you had your period?**
   - If **no**: direct to login, then to the website.
2. **If yes, ask more questions:**
   - Date of last period
   - How long the period lasted
   - Cycle length
   - If cycle length or period length isn't shared, assume a **28-day cycle** with a **5-day period**.
3. **Are you pregnant?**
   - *Open question: we're not sure yet what the significance of this question is, but we plan to have two modes: **pregnancy** and **period**.*
   - In **pregnancy mode**, the calendar does not display future period dates until a period is logged, so it works like a pause.

### 2. Login

- Credentials collected at signup and login: **username, password, and email**.
- The email is used for verification.
- A note will say: *please do not set your username as your actual government name.*
  - We aren't going to do anything with users' data, but the note is added for privacy.

### 3. Main Interface (Home)

- **Navigation bar:** Home, Tutorials, Chatbot, Shopping, Partners
- The calendar with dates marked.
- The **log period** and **log symptoms** forms sit below the calendar.
- Date prediction does **not** use ML. It uses a formula, to reduce the complexity of building the application.

### 4. Tutorials

- Contains the basic things we think users need to be educated on (see Basic Functionality above).

### 5. Chatbot

- RAG-based chatbot (useful both before period, during period and during pregnancy, also for partners)

### 6. Shopping

- This is the unique feature of the app.
- The page lists items we think the user should keep ready before or during their period.
- We are not the direct providers. Instead, we link to third-party delivery services.

### 7. Partners

- Inspired by Flo's partner feature.
- A user can share their cycle info with **one partner at a time**.
- The user can **customize what information is shared**:
  - **Base level (always shared):** period dates and ovulation dates (if we implement ovulation).
  - **Customizable:** other things, such as symptoms (the user chooses which symptoms to share).
- The partner's interface shows **how she might be feeling** and **what the partner should do** to make her feel better.