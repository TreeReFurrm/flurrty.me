# flurrty.me
[AI_TRAINING_SYSTEM_README.md](https://github.com/user-attachments/files/23614661/AI_TRAINING_SYSTEM_README.md)
# AI Training System - AI Twin Creation

## Overview
The AI training system allows users to upload their photos and train a personalized AI model that creates their "AI Flurrty Twin" - a custom AI-generated character based on their actual appearance.

## Architecture

### Database Schema
**ai_training_jobs** table:
- `id`: UUID primary key
- `user_id`: Reference to auth.users
- `furrsona_name`: Name of the Flurrty Twin
- `status`: pending, uploading, training, completed, failed
- `progress`: 0-100 percentage
- `estimated_completion`: Timestamp
- `replicate_training_id`: Replicate API training ID
- `replicate_model_version`: Trained model version ID
- `error_message`: Error details if failed
- `photo_urls`: Array of storage URLs
- `customization_data`: JSON with fur type, colors, style, personality, voice
- `created_at`, `completed_at`, `updated_at`

### Storage
**training-photos** bucket: Private storage for user uploaded photos

### Edge Functions

#### 1. upload-training-photos
Handles photo uploads to Supabase storage and creates training job record.

**Input:**
- FormData with photos, fursonaName, userId, customizationData

**Output:**
- jobId, photoUrls

#### 2. start-ai-training
Uploads photos to Replicate and starts FLUX model fine-tuning.

**Input:**
- jobId, userId

**Process:**
1. Fetches job details from database
2. Downloads photos from Supabase storage
3. Creates zip file and uploads to Replicate Files API
4. Starts training using ostris/flux-dev-lora-trainer
5. Updates job with replicate_training_id

**Output:**
- trainingId

#### 3. check-training-status
Polls Replicate API for training status and updates database.

**Input:**
- jobId

**Process:**
1. Fetches job from database
2. Queries Replicate training status
3. Updates job status, progress, and model version
4. Handles completion and errors

**Output:**
- status, progress, modelVersion, errorMessage

## User Flow

1. **Upload Photos (Step 1)**
   - User uploads 3-6 photos
   - Photos stored temporarily in component state

2. **Customize Features (Step 2)**
   - Select type (Glamour, Cosplay, Business, etc.)
   - Pick hybrid style (Realistic, Anthro, Cartoon, etc.)

3. **Personality & Voice (Step 3)**
   - Describe personality traits
   - Select voice tone

4. **Preview & Create (Step 4)**
   - Name the Flurrty Twin
   - Review all selections
   - Click "Create My Flurrty Twin"

5. **Training Process**
   - Photos uploaded to storage (20% progress)
   - Training job created in database
   - AI training started on Replicate (30% progress)
   - Status polled every 10 seconds
   - Completion at 100%

6. **Completion**
   - User notified when ready
   - Model version stored in database
   - Can be used for generating content

## Training Costs
- Runs on Nvidia H100 GPU
- ~$0.001528 per second
- Typical training: 8-10 minutes
- Cost: ~$1-2 per training

## Error Handling
- Upload failures: Retry mechanism
- Training failures: Error message stored
- Network issues: Status polling continues
- User notified of all errors

## Future Enhancements
- Email notifications on completion
- Webhook integration for real-time updates
- Batch training for multiple users
- Training progress visualization
- Model versioning and rollback

