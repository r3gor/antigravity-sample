# Quizz App - API

The AI Development Quiz App is an educational product designed to help users test and reinforce their understanding of AI software development concepts, including agent design, prompt engineering, and workflow automation.

This project is a RESTful API to manage quizzes, track user attempts, and calculate scores. This backend service focuses exclusively on server-side logic, data persistence, API design, and asynchronous service communication.

## Requirements

### Required API Endpoints

#### Quiz Management
- Retrieve all available quizzes (id, title, description only)
- Get full quiz details, including all questions and options (but NOT correct answers)
- Create a new quiz

#### Quiz Attempt & Submission
- Start a new quiz attempt for a user
- Submit answers and get results with scoring and feedback. Provide contextual performance feedback based on the user's score (e.g., 80%+ encouraging messages, 60-79% motivational feedback, <60% encouragement to improve)
- Trigger asynchronous email notification when a quiz is completed, sending the user a results summary

#### User Progress
- Get all quiz attempts for a specific user
- Get detailed results of a particular attempt (Attempt ID, User ID, Quiz ID and title, Timestamp, Overall score, and question-by-question breakdown with correctness and explanations)
- Get aggregate statistics for a user (total attempts, average score)

#### Persistence & Data
Your API should handle and store:
- Quiz information (title, description, questions)
- Questions with multiple choice options, correct answers, and explanations
- User identifiers and contact information (email)
- Attempt records (which user took which quiz, when, and their score)
- Individual answer submissions for each attempt
- Notification tracking (status, timestamps)

#### Email notificacions
- When a user completes a quiz, the system must asynchronously communicate with an email notification
service to send a results summary
- The email service must be mock for now
- Handle the scenario where the quiz submission should not fail if the email service is unavailable
- Track notification status appropriately
- Email notification should include:
    - User's name and email
    - Quiz title
    - Score (correct answers / total questions)
    - Percentage score
    - Performance feedback message
    - Timestamp of completion

## Example Scenario

API Usage Example:

1. A user sends a request to retrieve all available quiz categories from the API
2. The API returns three quiz categories. The user selects "Agent Fundamentals"
3. The user sends a request to start a new quiz attempt for the "Agent Fundamentals" quiz
4. The API responds with the attempt details and all 5 questions with their options (but not the correct
answers)
5. The user reviews the questions and prepares answers for all 5 questions
6. The user submits the answers to the API
7. The API processes the answers, calculates a score of 4/5 (80%), and returns:
    - Overall score and percentage
    - Performance feedback: "Good job! You're getting there!"
    - For each question: correct/incorrect status and explanation
    - The API asynchronously triggers a notification to send the user an email with the results
8. The email notification service (mocked) processes the request and simulates sending the email
9. The user sends a request to view quiz attempt history
10. The user decides to retake the quiz and sends another request to start a new attempt for the same quiz
11. The API creates a new attempt record while preserving the previous attempt history