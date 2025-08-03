# 🚀 Supabase Integration for AETHERIC AI

## Overview
AETHERIC AI now uses **Supabase** as the primary database and authentication solution, providing:
- ✅ **Persistent user profiles** across devices and sessions
- ✅ **1GB free cloud storage** for profile pictures
- ✅ **PostgreSQL database** with real-time capabilities
- ✅ **Robust authentication** (Email/Password, Google Sign-in)
- ✅ **No billing required** - completely free tier
- ✅ **Real-time subscriptions** and automatic backups

## Why Supabase?
- **Better than Firebase**: No billing requirements for storage
- **Generous free tier**: 1GB storage, 500MB database, unlimited auth
- **Modern PostgreSQL**: More powerful than NoSQL alternatives
- **Real-time**: Built-in real-time subscriptions
- **Open source**: Full transparency and control

## Architecture
- **Authentication**: Supabase Auth (Email/Password, Google)
- **Database**: PostgreSQL (user profiles, settings)
- **Storage**: Supabase Storage (profile pictures)
- **AI Chat**: Puter.com (unchanged)

## Quick Setup (10 minutes)

### 1. Create Supabase Project
1. Visit [Supabase.com](https://supabase.com/)
2. Click **"Start your project"**
3. Sign in with GitHub/Google
4. Click **"New project"**
5. Enter project name: `aetheric-ai`
6. Set a strong database password
7. Choose your preferred region
8. Click **"Create new project"**

### 2. Set Up Database Tables
1. Go to **SQL Editor** in your Supabase dashboard
2. Click **"New query"**
3. Copy and paste this SQL:

```sql
-- User profiles table
CREATE TABLE user_profiles (
    id UUID REFERENCES auth.users(id) PRIMARY KEY,
    email TEXT,
    username TEXT,
    display_name TEXT,
    bio TEXT,
    location TEXT,
    age TEXT,
    profile_image_url TEXT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create user_settings table
CREATE TABLE user_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES user_profiles(id) ON DELETE CASCADE,
    theme VARCHAR(20) DEFAULT 'system',
    language VARCHAR(10) DEFAULT 'en',
    notifications BOOLEAN DEFAULT true,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create conversations table for chat session persistence
CREATE TABLE conversations (
    id TEXT PRIMARY KEY,
    user_id UUID REFERENCES user_profiles(id) ON DELETE CASCADE,
    title TEXT NOT NULL DEFAULT 'New Chat',
    messages JSONB NOT NULL DEFAULT '[]',
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE user_settings ENABLE ROW LEVEL SECURITY;
ALTER TABLE conversations ENABLE ROW LEVEL SECURITY;

-- Create policies for user_profiles
CREATE POLICY "Users can view own profile" ON user_profiles
    FOR SELECT USING (auth.uid() = id);

CREATE POLICY "Users can update own profile" ON user_profiles
    FOR UPDATE USING (auth.uid() = id);

CREATE POLICY "Users can insert own profile" ON user_profiles
    FOR INSERT WITH CHECK (auth.uid() = id);

-- Create policies for user_settings
CREATE POLICY "Users can view own settings" ON user_settings
    FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can update own settings" ON user_settings
    FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own settings" ON user_settings
    FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Create policies for conversations
CREATE POLICY "Users can view own conversations" ON conversations
    FOR SELECT USING (user_id = auth.uid());

CREATE POLICY "Users can insert own conversations" ON conversations
    FOR INSERT WITH CHECK (user_id = auth.uid());

CREATE POLICY "Users can update own conversations" ON conversations
    FOR UPDATE USING (user_id = auth.uid());

CREATE POLICY "Users can delete own conversations" ON conversations
    FOR DELETE USING (user_id = auth.uid());
```

4. Click **"Run"** to execute the SQL

### 3. Set Up Storage
1. Go to **Storage** in your Supabase dashboard
2. Click **"Create a new bucket"**
3. Enter bucket name: `profile-images`
4. Make it **Public**: Yes
5. Click **"Create bucket"**

### 4. Set Storage Policies
1. Go to **Storage** → **profile-images** → **Policies**
2. Click **"Add policy"** and create these policies:

**Policy 1: Upload Policy**
```sql
CREATE POLICY "Users can upload own images" ON storage.objects
    FOR INSERT WITH CHECK (
        bucket_id = 'profile-images' AND
        auth.uid()::text = (storage.foldername(name))[1]
    );
```

**Policy 2: View Policy**
```sql
CREATE POLICY "Public can view images" ON storage.objects
    FOR SELECT USING (bucket_id = 'profile-images');
```

**Policy 3: Delete Policy**
```sql
CREATE POLICY "Users can delete own images" ON storage.objects
    FOR DELETE USING (
        bucket_id = 'profile-images' AND
        auth.uid()::text = (storage.foldername(name))[1]
    );
```

### 5. Enable Authentication
1. Go to **Authentication** → **Settings**
2. Enable **Email** authentication
3. (Optional) Enable **Google OAuth**:
   - Go to **Authentication** → **Providers**
   - Enable Google
   - Add your Google OAuth credentials
4. Set **Site URL** to your domain (e.g., `http://localhost:3000`)

### 6. Get Project Credentials
1. Go to **Settings** → **API**
2. Copy the **Project URL**
3. Copy the **anon public** key
4. Update `src/config/supabase-config.js`:

```javascript
export const supabaseConfig = {
    url: 'https://your-project-ref.supabase.co',
    anonKey: 'your-anon-key-here'
};
```

## Features

### 🔐 Authentication
- **Email/Password**: Traditional sign-up/sign-in
- **Google Sign-in**: One-click authentication
- **Persistent sessions**: Stay logged in across browser sessions
- **Secure**: Industry-standard authentication with JWT tokens

### 👤 User Profiles
- **Complete profiles**: Username, email, bio, location, age
- **Profile pictures**: Uploaded to Supabase Storage
- **Persistent**: Data saved across all devices
- **Real-time**: Instant updates with subscriptions

### 📁 Cloud Storage
- **Profile pictures**: Stored in Supabase Storage
- **1GB free**: Generous free tier storage
- **Fast**: Global CDN delivery
- **Secure**: Row-level security and access control

### 💾 Database Structure
```
user_profiles
├── id: UUID (Primary Key, references auth.users)
├── email: TEXT
├── username: TEXT
├── display_name: TEXT
├── bio: TEXT
├── location: TEXT
├── age: TEXT
├── profile_image_url: TEXT (Supabase Storage URL)
├── created_at: TIMESTAMP
└── updated_at: TIMESTAMP

user_settings
├── user_id: UUID (Primary Key, references auth.users)
├── theme: TEXT
├── language: TEXT
├── notifications: BOOLEAN
├── created_at: TIMESTAMP
└── updated_at: TIMESTAMP
```

## Free Tier Limits
Supabase offers excellent free limits:
- **Database**: 500MB PostgreSQL database
- **Storage**: 1GB file storage
- **Bandwidth**: 2GB/month
- **Authentication**: Unlimited users
- **API requests**: 50,000/month
- **Real-time**: Unlimited connections

## Migration from Firebase/Puter.com
The app automatically handles the migration:
1. **New users**: Automatically use Supabase
2. **Existing users**: Data will be re-created in Supabase on next sign-in
3. **Fallback**: localStorage backup for offline functionality

## Real-time Features
Supabase provides real-time subscriptions:
```javascript
// Subscribe to profile changes
const subscription = supabase
    .channel('profile-changes')
    .on('postgres_changes', {
        event: '*',
        schema: 'public',
        table: 'user_profiles',
        filter: `id=eq.${userId}`
    }, (payload) => {
        console.log('Profile updated:', payload);
    })
    .subscribe();
```

## Troubleshooting

### Common Issues
1. **"Invalid API key"**
   - Check that `supabase-config.js` has correct URL and key
   - Ensure you're using the `anon` key, not the `service_role` key

2. **"Row Level Security policy violation"**
   - Check that RLS policies are set up correctly
   - Ensure user is authenticated before accessing data

3. **Profile picture not uploading**
   - Check Storage bucket is created and public
   - Verify storage policies are set up
   - Check file size < 50MB (Supabase limit)

4. **"relation does not exist"**
   - Ensure database tables are created using the SQL above
   - Check table names match exactly (case-sensitive)

### Debug Mode
Enable debug logging in your browser console to see detailed Supabase operations.

## Security Best Practices
- **Row Level Security**: Enabled by default
- **API Keys**: Never expose `service_role` key in frontend
- **Policies**: Users can only access their own data
- **HTTPS**: All communication encrypted
- **JWT**: Secure token-based authentication

## Support
- **Supabase Documentation**: https://supabase.com/docs
- **Dashboard**: https://supabase.com/dashboard
- **Community**: https://github.com/supabase/supabase/discussions

---

🎉 **Your AETHERIC AI app now has enterprise-grade PostgreSQL database with 1GB free storage!**

## Next Steps
1. Create your Supabase project
2. Run the SQL commands to set up tables
3. Create the storage bucket
4. Update your configuration
5. Test profile picture uploads

Your profile pictures will now persist forever across all devices and sessions!
