# DermPal Normalisation
Normalisation is a technique in organising data in a relational database to reduce redundancy and improve integrity in data. This is done by breaking down large tables into smaller tables that eliminate anomalies and ensures data consistency.

## Starting record: Unnormalised data

An unnormalised table may store several images, predictions and recommendations inside one assessment record.

For example:

UNNORMALISED_ASSESSMENT

| assessment_id | user_id | user_name  | images               | predictions                     | recommendations                       |
| ------------- | ------- | ---------- | -------------------- | ------------------------------- | ------------------------------------- |
| A001          | U001    | Ada James  | front.jpg, cheek.jpg | skin_type: oily; acne: moderate | Oil-free moisturiser; gentle cleanser |
| A002          | U002    | David Cole | forehead.jpg         | skin_type: dry; acne: mild      | Hydrating moisturiser                 |

The problem is that several values are stored inside individual fields.

For example, the `images` field for A001 contains two filenames. The `predictions` field also contains multiple results.

This creates problems:

- Searching for a particular image becomes difficult.
- Updating one prediction may require changing a larger text field.
- Adding another image or prediction makes the record increasingly complicated.
- The same user's details may be repeated across multiple assessments.

The solution is to organise the data into tables containing individual values.

## 3. First Normal Form (1NF)

A table satisfies First Normal Form when:

- Each column contains one value per record.
- There are no lists or repeating groups inside a field.
- Each record can be uniquely identified.

For DermPal, each assessment image becomes an individual row.

ASSESSMENT_IMAGE_1NF

| assessment_id | image_number | submitted_at | user_id | user_name  | file_name    |
| ------------- | ------------ | ------------ | ------- | ---------- | ------------ |
| A001          | 1            | 2026-10-20   | U001    | Ada James  | front.jpg    |
| A001          | 2            | 2026-10-20   | U001    | Ada James  | cheek.jpg    |
| A002          | 1            | 2026-10-21   | U002    | David Cole | forehead.jpg |

Primary key: `(assessment_id, image_number)`

A primary key is a field, or combination of fields, that uniquely identifies each record.

The combination of `assessment_id` and `image_number` identifies each image within an assessment.

### What changes in 1NF?

The repeated image list is removed. Each image now occupies its own row, making individual images easier to identify and manage.

However, the table still repeats assessment and user information. For example, Ada's name and assessment submission date appear twice.

These repeated details can create update problems. If Ada's name changes, every affected row must be updated.

This leads to Second Normal Form.

## 4. Second Normal Form (2NF)

A table satisfies Second Normal Form when:

- It already satisfies 1NF.
- Every non-key field depends on the whole primary key.

A partial dependency occurs when a non-key field depends on only part of a combined primary key.

In the 1NF example, the primary key is:

`(assessment_id, image_number)`

However:

- `user_id`, `user_name` and `submitted_at` depend on `assessment_id` alone.
- `file_name` depends on the individual image identified by the complete combination.

Therefore, assessment information is repeated across image records and does not depend on the entire combined key.

### Separating the tables

The information is divided into two tables.

ASSESSMENT

| assessment_id (PK) | user_id | submitted_at |
| ------------------ | ------- | ------------ |
| A001               | U001    | 2026-10-20   |
| A002               | U002    | 2026-10-21   |

ASSESSMENT_IMAGE

| image_id (PK) | assessment_id (FK) | file_name    |
| ------------- | ------------------ | ------------ |
| I001          | A001               | front.jpg    |
| I002          | A001               | cheek.jpg    |
| I003          | A002               | forehead.jpg |

PK means Primary Key. FK means Foreign Key: a field that references the primary key in another table.

### What changes in 2NF?

The assessment information moves into its own table, while image details remain in `ASSESSMENT_IMAGE`.

This removes the partial dependency because:

- Assessment details depend on `assessment_id`.
- Image details depend on the individual image record.

The `assessment_id` foreign key keeps the two tables connected.

This also reduces repetition: the assessment submission date no longer needs to be repeated for every image.

### What about the other DermPal tables?

The same principle applies to the rest of the design.

For example, a prediction belongs to one assessment and one model version. Its record is identified by `prediction_id`, so its fields depend on that complete key.

The `MODEL_VERSION` table independently stores model details, while `RECOMMENDATION` independently stores recommendation records.

## 5. Third Normal Form (3NF)

A table satisfies Third Normal Form when:

- It already satisfies 2NF.
- Non-key fields depend on the primary key, not on another non-key field.

This removes transitive dependencies.

A transitive dependency occurs when a non-key field determines another non-key field.

Consider the `ASSESSMENT` information before separating user details:

ASSESSMENT_2NF_EXTENDED

| assessment_id (PK) | user_id | user_name  | user_email         | submitted_at |
| ------------------ | ------- | ---------- | ------------------ | ------------ |
| A001               | U001    | Ada James  | ada\@example.com   | 2026-10-20   |
| A002               | U002    | David Cole | david\@example.com | 2026-10-21   |

Here:

- `assessment_id` determines `user_id`.
- `user_id` determines `user_name` and `user_email`.

Therefore, user details depend on another non-key field (`user_id`) rather than directly on the assessment's primary key.

This creates unnecessary repetition if a user creates several assessments.

### Separating user information

The solution is to move user details into a separate `USER` table.

USER

| user_id (PK) | full_name  | email              |
| ------------ | ---------- | ------------------ |
| U001         | Ada James  | ada\@example.com   |
| U002         | David Cole | david\@example.com |

ASSESSMENT

| assessment_id (PK) | user_id (FK) | submitted_at |
| ------------------ | ------------ | ------------ |
| A001               | U001         | 2026-10-20   |
| A002               | U002         | 2026-10-21   |

Now:

- User details depend on `user_id`.
- Assessment details depend on `assessment_id`.
- The `user_id` foreign key connects each assessment to its owner.

This removes the transitive dependency and prevents user details from being unnecessarily repeated.

## 6. Final DermPal database tables (3NF)

After applying normalisation, the final design contains six related tables.

The following tables show selected fields and fictional example records for readability.

### 1. USER

Stores registered user information.

| user_id (PK) | full_name  | email              | role |
| ------------ | ---------- | ------------------ | ---- |
| U001         | Ada James  | ada\@example.com   | user |
| U002         | David Cole | david\@example.com | user |

### 2. ASSESSMENT

Stores each skin assessment submitted by a user.

| assessment_id (PK) | user_id (FK) | status    |
| ------------------ | ------------ | --------- |
| A001               | U001         | completed |
| A002               | U002         | completed |

### 3. ASSESSMENT_IMAGE

Stores image metadata and the reference to the externally stored image file.

| image_id (PK) | assessment_id (FK) | file_name    |
| ------------- | ------------------ | ------------ |
| I001          | A001               | front.jpg    |
| I002          | A001               | cheek.jpg    |
| I003          | A002               | forehead.jpg |

### 4. MODEL_VERSION

Stores the AI model names, types and versions.

| model_version_id (PK) | model_name     | model_type | version |
| --------------------- | -------------- | ---------- | ------- |
| M001                  | SkinClassifier | skin_type  | 1.0     |
| M002                  | AcneDetector   | acne       | 1.2     |

### 5. PREDICTION

Stores the AI-generated skin-type and acne prediction results.

| prediction_id (PK) | assessment_id (FK) | model_version_id (FK) | prediction_type | predicted_class |
| ------------------ | ------------------ | --------------------- | --------------- | --------------- |
| P001               | A001               | M001                  | skin_type       | oily            |
| P002               | A001               | M002                  | acne            | moderate        |
| P003               | A002               | M001                  | skin_type       | dry             |
| P004               | A002               | M002                  | acne            | mild            |

### 6. RECOMMENDATION

Stores skincare recommendations associated with prediction results.

| recommendation_id (PK) | prediction_id (FK) | recommendation_text           |
| ---------------------- | ------------------ | ----------------------------- |
| R001                   | P001               | Use an oil-free moisturiser   |
| R002                   | P002               | Use a gentle acne cleanser    |
| R003                   | P003               | Use a hydrating moisturiser   |
| R004                   | P004               | Use a gentle skincare routine |

### How the final tables remain connected

The foreign keys maintain the relationships between the six tables:

- `ASSESSMENT.user_id` references `USER.user_id`.
- `ASSESSMENT_IMAGE.assessment_id` references `ASSESSMENT.assessment_id`.
- `PREDICTION.assessment_id` references `ASSESSMENT.assessment_id`.
- `PREDICTION.model_version_id` references `MODEL_VERSION.model_version_id`.
- `RECOMMENDATION.prediction_id` references `PREDICTION.prediction_id`.

The final design uses automatically generated identifiers such as `image_id` instead of the illustrative combined key `(assessment_id, image_number)`. This makes each image independently identifiable and matches the agreed DermPal design.

An automatically generated ID does not, by itself, guarantee 3NF. The design still needs to ensure that non-key attributes depend on the correct keys and that transitive dependencies are removed.

Additional fields omitted from the example tables for readability:

| Table            | Omitted fields                                     |
| ---------------- | -------------------------------------------------- |
| USER             | `created_at`                                       |
| ASSESSMENT       | `submitted_at`                                     |
| ASSESSMENT_IMAGE | `storage_reference`, `content_type`, `uploaded_at` |
| MODEL_VERSION    | `created_at`                                       |
| PREDICTION       | `confidence_score`, `predicted_at`                 |
| RECOMMENDATION   | `created_at`                                       |

The final tables also retain the agreed constraints, including unique user emails, unique model name and version combinations, valid prediction types, confidence scores between 0 and 1, and prevention of duplicate prediction types for the same assessment and model version.

## 7. Summary of normalisation

| Normalisation stage | What happens in DermPal                                                                          | Problem addressed                                   |
| ------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| Unnormalised        | Images, predictions and recommendations appear as lists in records.                              | Repeating groups and difficult updates.             |
| 1NF                 | Each image is represented by an individual row.                                                  | Multiple values in one field.                       |
| 2NF                 | Assessment information and image information are separated.                                      | Partial dependencies on a combined key.             |
| 3NF                 | User information is separated from assessment information; other entities retain their own data. | Transitive dependencies and unnecessary repetition. |

Conclusion

The DermPal database design follows 1NF, 2NF and 3NF by separating user information, assessments, uploaded image metadata, AI model versions, prediction results and recommendations into related tables.

This structure reduces data duplication, improves consistency, supports reliable updates and maintains relationships through primary and foreign keys. It also provides a practical relational foundation for the DermPal platform while keeping the actual image files in external storage.