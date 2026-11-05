# Portfolio 1 Database Design and Modelling

## Case study

DermPal is a simulated AI-powered skin assessment platform that enables users to upload one or more photographs of their skin for automated skin-type classification and acne analysis. When a user submits images, the system creates an assessment record to track the submission and its results. The database stores user information, assessment records, image metadata and storage references, AI-generated prediction results, model versions and skincare recommendations. It provides a structured way to manage image submissions, track prediction results and retrieve users' assessment histories.

## Users

The system serves registered users who upload skin images for analysis and view their assessment histories and prediction results; administrators who manage platform records and monitor system activities; and reviewing professionals who examine assessment results and associated recommendations.

## Main operations

The system supports user registration, uploading one or more skin images for analysis, creating assessment records for image submissions, recording uploaded image metadata and storage references, storing AI-generated skin-type and acne predictions, retrieving individual assessment histories, managing AI model versions, and generating and storing skincare recommendations based on prediction results.

## Business rules

1. Each user must have a unique user ID and email address. A user may create multiple skin assessments.
2. Each assessment must have a unique assessment ID and belong to exactly one registered user.
3. Each assessment must contain one or more image records, and every image record must belong to an existing assessment.
4. Each AI prediction must have a unique prediction ID and reference one existing assessment and one registered model version.
5. Prediction types must be restricted to `skin_type` or `acne`, with predicted classes corresponding to the selected prediction type.
6. Each prediction must have a confidence score between 0 and 1, inclusive, and a recorded prediction timestamp.
7. An assessment must not have duplicate predictions of the same type from the same model version, preventing conflicting duplicate results.
8. Recommendations must reference an existing prediction and may only be generated for assessments with completed predictions.

## Scope

The database covers user records, skin assessments, submitted image metadata and storage references, AI model versions, prediction results and associated skincare recommendations using fictional data. Actual image files will be stored outside the relational database in external object storage, while their metadata and storage references will be maintained in the database. The scope excludes user authentication implementation, payment processing, live AI model execution, medical diagnosis, appointment booking and communication with external healthcare providers.
