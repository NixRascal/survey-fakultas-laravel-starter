flowchart TD
  Start[Start] --> Choice{User Type}
  Choice -->|Admin| AdminSignIn[Admin Sign In]
  Choice -->|Respondent| SurveyList[List Active Surveys]
  AdminSignIn --> Dashboard[Dashboard]
  Dashboard --> ManageSurveys[Manage Surveys]
  ManageSurveys --> CreateSurvey[Create Survey]
  CreateSurvey --> AddQuestions[Add Questions]
  Dashboard --> ViewReports[View Reports]
  ViewReports --> ExportPDF[Export PDF]
  SurveyList --> TakeSurvey[Take Survey]
  TakeSurvey --> SubmitResponse[Submit Response]
  SubmitResponse --> ThankYou[Thank You]