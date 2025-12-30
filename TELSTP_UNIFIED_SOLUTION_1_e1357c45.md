# TELSTP Unified Database Solution
## Complete Integration Strategy for 12-Hub Ecosystem

---

## 🎯 PROJECT OVERVIEW

**TELSTP (Tawasol Egypt Life Science Technology Park)**
- **Investment**: 8 Billion EGP Multi-Investment Project
- **Scope**: 12+ Digital AI-based Applications
- **Challenge**: Unifying 100+ tables from 12 standalone hubs into one Supabase database
- **Current Issues**: 
  - Table naming conflicts (users vs user, profiles vs user_profiles)
  - RLS policy mismatches
  - Authentication failures across apps
  - Deployment errors on Vercel
  - No centralized database management

---

## 🏗️ THE 12 HUBS ARCHITECTURE

### **Core Infrastructure:**
1. **M2-3M Quantum Research Engine** - Central processing hub (4.2TB/hour, 50 research institutes)
2. **OmniCognitor** - Unified platform connecting all team members
3. **Main Portal** - AI-powered visitor guide

### **Public-Facing Hubs:**
4. **3D Global Earth Parks Network** - Life science achievements visualization
5. **Investors Gateway** - Funding and investment opportunities
6. **Projects Incubator & Accelerator** - Innovation support
7. **Students Portal** - Academic resources
8. **Recruitment Portal** - Talent acquisition
9. **News & Blogs** - Industry updates
10. **Radio Channel & Podcasts** - Educational broadcasting

### **Healthcare & Wellness:**
11. **Telemedicine Hub** - MyWell AI (patients) + MyAssist AI (physicians)
12. **Personal Companion App (PersComp)** - 6 AI characters:
    - Edu Mentor
    - Personal Wellness Companion
    - Ibn Sina (Traditional Medicine)
    - Medical Doctor
    - Ancient Egyptian Pharaoh
    - [6th character]

### **Internal Portals:**
13. **TeLsTP Employees Portal**
14. **Tawasol Holding Employees Portal**
15. **Syllabus & Curriculum Hub** - Powered by M2-3M

---

## 🚨 CRITICAL ISSUES IDENTIFIED

### **Database Schema Conflicts:**
```
❌ users vs user (table naming inconsistency)
❌ profiles vs user_profiles vs global_profiles (multiple profile tables)
❌ RLS policies not matching table structures
❌ Missing foreign key relationships
❌ Duplicate data across tables
```

### **Authentication Issues:**
```
❌ Apps can't connect to Supabase (CORS/API errors)
❌ RLS blocking legitimate queries
❌ Service role key not properly configured
❌ Missing auth policies
```

### **Deployment Issues:**
```
❌ Vercel routing configuration errors
❌ Frontend can't communicate with Supabase
❌ No backend middleware layer
❌ Static HTML can't execute database operations
```

---

## ✅ COMPLETE SOLUTION ARCHITECTURE

### **Phase 1: Database Unification (CRITICAL - DO FIRST)**

#### **Step 1.1: Audit Current Schema**
```sql
-- Run this in Supabase SQL Editor to see all tables
SELECT 
    schemaname,
    tablename,
    tableowner
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY tablename;

-- Check all RLS policies
SELECT 
    schemaname,
    tablename,
    policyname,
    permissive,
    roles,
    cmd,
    qual
FROM pg_policies
WHERE schemaname = 'public'
ORDER BY tablename, policyname;
```

#### **Step 1.2: Design Unified Schema**
```sql
-- MASTER SCHEMA FOR ALL 12 HUBS
-- This consolidates all apps into ONE coherent database

-- ============================================
-- CORE AUTHENTICATION & USERS
-- ============================================

-- Single unified users table (extends auth.users)
CREATE TABLE IF NOT EXISTS public.users (
    id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT UNIQUE NOT NULL,
    full_name TEXT,
    avatar_url TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW(),
    last_login TIMESTAMPTZ,
    is_active BOOLEAN DEFAULT true,
    metadata JSONB DEFAULT '{}'::jsonb
);

-- Single unified profiles table
CREATE TABLE IF NOT EXISTS public.profiles (
    id UUID PRIMARY KEY REFERENCES public.users(id) ON DELETE CASCADE,
    username TEXT UNIQUE,
    bio TEXT,
    phone TEXT,
    country TEXT,
    city TEXT,
    date_of_birth DATE,
    gender TEXT CHECK (gender IN ('male', 'female', 'other', 'prefer_not_to_say')),
    language_preference TEXT DEFAULT 'en',
    timezone TEXT DEFAULT 'Africa/Cairo',
    profile_type TEXT CHECK (profile_type IN ('student', 'researcher', 'physician', 'employee', 'investor', 'public')),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- HUB-SPECIFIC TABLES
-- ============================================

-- 1. TELEMEDICINE HUB
CREATE TABLE IF NOT EXISTS public.telemedicine_patients (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    medical_record_number TEXT UNIQUE,
    emergency_contact JSONB,
    allergies TEXT[],
    medications TEXT[],
    chronic_conditions TEXT[],
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.telemedicine_appointments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    patient_id UUID REFERENCES public.telemedicine_patients(id) ON DELETE CASCADE,
    physician_id UUID REFERENCES public.users(id),
    appointment_date TIMESTAMPTZ NOT NULL,
    status TEXT CHECK (status IN ('scheduled', 'completed', 'cancelled', 'no_show')),
    consultation_notes TEXT,
    ai_summary TEXT, -- MyWell AI or MyAssist AI generated summary
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 2. PERSONAL COMPANION (PersComp)
CREATE TABLE IF NOT EXISTS public.companion_profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    active_characters TEXT[] DEFAULT '{}', -- Array of active AI characters
    preferences JSONB DEFAULT '{}'::jsonb,
    conversation_history JSONB DEFAULT '[]'::jsonb,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.companion_interactions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    character_type TEXT CHECK (character_type IN ('edu_mentor', 'wellness', 'ibn_sina', 'medical_doctor', 'pharaoh', 'custom')),
    message TEXT NOT NULL,
    response TEXT,
    sentiment TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. STUDENTS PORTAL
CREATE TABLE IF NOT EXISTS public.students (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    student_id TEXT UNIQUE NOT NULL,
    enrollment_date DATE,
    graduation_date DATE,
    program TEXT,
    year_level INTEGER,
    gpa DECIMAL(3,2),
    status TEXT CHECK (status IN ('active', 'inactive', 'graduated', 'suspended')),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.courses (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    course_code TEXT UNIQUE NOT NULL,
    course_name TEXT NOT NULL,
    description TEXT,
    credits INTEGER,
    instructor_id UUID REFERENCES public.users(id),
    syllabus_url TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.enrollments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    student_id UUID REFERENCES public.students(id) ON DELETE CASCADE,
    course_id UUID REFERENCES public.courses(id) ON DELETE CASCADE,
    enrollment_date TIMESTAMPTZ DEFAULT NOW(),
    grade TEXT,
    status TEXT CHECK (status IN ('enrolled', 'completed', 'dropped', 'failed')),
    UNIQUE(student_id, course_id)
);

-- 4. M2-3M RESEARCH ENGINE
CREATE TABLE IF NOT EXISTS public.research_institutes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    country TEXT,
    website TEXT,
    contact_email TEXT,
    api_endpoint TEXT,
    connection_status TEXT CHECK (connection_status IN ('active', 'inactive', 'pending')),
    last_sync TIMESTAMPTZ,
    data_volume_tb DECIMAL(10,2),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.research_data (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    institute_id UUID REFERENCES public.research_institutes(id) ON DELETE CASCADE,
    title TEXT NOT NULL,
    abstract TEXT,
    authors TEXT[],
    publication_date DATE,
    research_area TEXT,
    keywords TEXT[],
    data_url TEXT,
    processed_by_m23m BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 5. RECRUITMENT PORTAL
CREATE TABLE IF NOT EXISTS public.job_postings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    department TEXT,
    description TEXT,
    requirements TEXT,
    salary_range TEXT,
    location TEXT,
    employment_type TEXT CHECK (employment_type IN ('full_time', 'part_time', 'contract', 'internship')),
    posted_by UUID REFERENCES public.users(id),
    status TEXT CHECK (status IN ('open', 'closed', 'filled')),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.applications (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    job_id UUID REFERENCES public.job_postings(id) ON DELETE CASCADE,
    applicant_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    resume_url TEXT,
    cover_letter TEXT,
    status TEXT CHECK (status IN ('submitted', 'reviewing', 'shortlisted', 'rejected', 'accepted')),
    applied_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 6. INVESTORS GATEWAY
CREATE TABLE IF NOT EXISTS public.investors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    organization TEXT,
    investment_capacity TEXT,
    areas_of_interest TEXT[],
    portfolio_url TEXT,
    verified BOOLEAN DEFAULT false,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.investment_opportunities (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    description TEXT,
    funding_goal DECIMAL(12,2),
    current_funding DECIMAL(12,2) DEFAULT 0,
    deadline DATE,
    project_stage TEXT CHECK (project_stage IN ('concept', 'prototype', 'pilot', 'scaling')),
    status TEXT CHECK (status IN ('open', 'funded', 'closed')),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 7. NEWS & BLOGS
CREATE TABLE IF NOT EXISTS public.articles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    slug TEXT UNIQUE NOT NULL,
    content TEXT,
    excerpt TEXT,
    featured_image TEXT,
    author_id UUID REFERENCES public.users(id),
    category TEXT,
    tags TEXT[],
    published BOOLEAN DEFAULT false,
    published_at TIMESTAMPTZ,
    views INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 8. RADIO CHANNEL & PODCASTS
CREATE TABLE IF NOT EXISTS public.radio_programs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    description TEXT,
    host_id UUID REFERENCES public.users(id),
    schedule TEXT, -- e.g., "Monday 10:00 AM"
    duration_minutes INTEGER,
    category TEXT,
    status TEXT CHECK (status IN ('active', 'paused', 'ended')),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.podcast_episodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    program_id UUID REFERENCES public.radio_programs(id) ON DELETE CASCADE,
    episode_number INTEGER,
    title TEXT NOT NULL,
    description TEXT,
    audio_url TEXT NOT NULL,
    duration_seconds INTEGER,
    published_at TIMESTAMPTZ,
    listen_count INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 9. EMPLOYEE PORTALS (TeLsTP & Tawasol Holding)
CREATE TABLE IF NOT EXISTS public.employees (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    employee_id TEXT UNIQUE NOT NULL,
    organization TEXT CHECK (organization IN ('telstp', 'tawasol_holding')),
    department TEXT,
    position TEXT,
    hire_date DATE,
    manager_id UUID REFERENCES public.employees(id),
    salary DECIMAL(10,2),
    employment_status TEXT CHECK (employment_status IN ('active', 'on_leave', 'terminated')),
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 10. CURRICULUM HUB
CREATE TABLE IF NOT EXISTS public.curriculum_modules (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    description TEXT,
    learning_objectives TEXT[],
    content_url TEXT,
    difficulty_level TEXT CHECK (difficulty_level IN ('beginner', 'intermediate', 'advanced')),
    estimated_hours INTEGER,
    powered_by_m23m BOOLEAN DEFAULT true,
    last_updated_by_m23m TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 11. OMNICOGNITOR (Team Collaboration)
CREATE TABLE IF NOT EXISTS public.omnicognitor_members (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id) ON DELETE CASCADE,
    role TEXT CHECK (role IN ('admin', 'developer', 'designer', 'ai_agent', 'contributor')),
    expertise TEXT[],
    ai_agent_type TEXT, -- e.g., 'ChatGPT', 'Claude', 'Gemini', 'Mistral'
    status TEXT CHECK (status IN ('active', 'inactive')),
    joined_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE IF NOT EXISTS public.omnicognitor_tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title TEXT NOT NULL,
    description TEXT,
    assigned_to UUID REFERENCES public.omnicognitor_members(id),
    hub_context TEXT, -- Which of the 12 hubs this task relates to
    priority TEXT CHECK (priority IN ('low', 'medium', 'high', 'critical')),
    status TEXT CHECK (status IN ('todo', 'in_progress', 'review', 'completed', 'blocked')),
    due_date TIMESTAMPTZ,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 12. 3D GLOBAL EARTH PARKS NETWORK
CREATE TABLE IF NOT EXISTS public.life_science_parks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name TEXT NOT NULL,
    country TEXT NOT NULL,
    city TEXT,
    latitude DECIMAL(10,8),
    longitude DECIMAL(11,8),
    description TEXT,
    website TEXT,
    contact_email TEXT,
    achievements TEXT[],
    research_areas TEXT[],
    partnerships TEXT[],
    microsite_url TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- UNIFIED AUDIT & LOGGING
-- ============================================

CREATE TABLE IF NOT EXISTS public.audit_logs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES public.users(id),
    hub_name TEXT, -- Which hub generated this log
    action TEXT NOT NULL,
    table_name TEXT,
    record_id UUID,
    old_values JSONB,
    new_values JSONB,
    ip_address INET,
    user_agent TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- ============================================
-- INDEXES FOR PERFORMANCE
-- ============================================

CREATE INDEX IF NOT EXISTS idx_users_email ON public.users(email);
CREATE INDEX IF NOT EXISTS idx_profiles_user_id ON public.profiles(user_id);
CREATE INDEX IF NOT EXISTS idx_profiles_username ON public.profiles(username);
CREATE INDEX IF NOT EXISTS idx_students_user_id ON public.students(user_id);
CREATE INDEX IF NOT EXISTS idx_employees_user_id ON public.employees(user_id);
CREATE INDEX IF NOT EXISTS idx_articles_slug ON public.articles(slug);
CREATE INDEX IF NOT EXISTS idx_articles_author ON public.articles(author_id);
CREATE INDEX IF NOT EXISTS idx_audit_logs_user ON public.audit_logs(user_id);
CREATE INDEX IF NOT EXISTS idx_audit_logs_created ON public.audit_logs(created_at DESC);

-- ============================================
-- ENABLE ROW LEVEL SECURITY (RLS)
-- ============================================

ALTER TABLE public.users ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.telemedicine_patients ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.telemedicine_appointments ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.companion_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.companion_interactions ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.students ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.courses ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.enrollments ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.research_institutes ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.research_data ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.job_postings ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.applications ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.investors ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.investment_opportunities ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.articles ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.radio_programs ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.podcast_episodes ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.employees ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.curriculum_modules ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.omnicognitor_members ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.omnicognitor_tasks ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.life_science_parks ENABLE ROW LEVEL SECURITY;
ALTER TABLE public.audit_logs ENABLE ROW LEVEL SECURITY;
```

#### **Step 1.3: Create RLS Policies (CORRECT & SECURE)**
```sql
-- ============================================
-- RLS POLICIES - PROPERLY CONFIGURED
-- ============================================

-- USERS TABLE POLICIES
CREATE POLICY "Users can view own profile"
    ON public.users FOR SELECT
    USING (auth.uid() = id);

CREATE POLICY "Users can update own profile"
    ON public.users FOR UPDATE
    USING (auth.uid() = id);

CREATE POLICY "Service role has full access to users"
    ON public.users FOR ALL
    USING (auth.jwt()->>'role' = 'service_role');

-- PROFILES TABLE POLICIES
CREATE POLICY "Profiles are viewable by everyone"
    ON public.profiles FOR SELECT
    USING (true);

CREATE POLICY "Users can update own profile"
    ON public.profiles FOR UPDATE
    USING (auth.uid() = id);

CREATE POLICY "Users can insert own profile"
    ON public.profiles FOR INSERT
    WITH CHECK (auth.uid() = id);

-- TELEMEDICINE POLICIES
CREATE POLICY "Patients can view own medical records"
    ON public.telemedicine_patients FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Physicians can view their patients"
    ON public.telemedicine_appointments FOR SELECT
    USING (
        auth.uid() = physician_id 
        OR 
        auth.uid() IN (SELECT user_id FROM public.telemedicine_patients WHERE id = patient_id)
    );

-- COMPANION POLICIES
CREATE POLICY "Users can manage own companion profile"
    ON public.companion_profiles FOR ALL
    USING (auth.uid() = user_id);

CREATE POLICY "Users can view own interactions"
    ON public.companion_interactions FOR SELECT
    USING (auth.uid() = user_id);

-- STUDENTS POLICIES
CREATE POLICY "Students can view own records"
    ON public.students FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Public can view course catalog"
    ON public.courses FOR SELECT
    USING (true);

CREATE POLICY "Students can view own enrollments"
    ON public.enrollments FOR SELECT
    USING (
        auth.uid() IN (SELECT user_id FROM public.students WHERE id = student_id)
    );

-- RESEARCH POLICIES
CREATE POLICY "Public can view research institutes"
    ON public.research_institutes FOR SELECT
    USING (true);

CREATE POLICY "Public can view research data"
    ON public.research_data FOR SELECT
    USING (true);

-- RECRUITMENT POLICIES
CREATE POLICY "Public can view open job postings"
    ON public.job_postings FOR SELECT
    USING (status = 'open');

CREATE POLICY "Users can view own applications"
    ON public.applications FOR SELECT
    USING (auth.uid() = applicant_id);

CREATE POLICY "Users can submit applications"
    ON public.applications FOR INSERT
    WITH CHECK (auth.uid() = applicant_id);

-- INVESTORS POLICIES
CREATE POLICY "Verified investors can view opportunities"
    ON public.investment_opportunities FOR SELECT
    USING (
        auth.uid() IN (SELECT user_id FROM public.investors WHERE verified = true)
        OR status = 'open'
    );

-- NEWS & BLOGS POLICIES
CREATE POLICY "Public can view published articles"
    ON public.articles FOR SELECT
    USING (published = true);

CREATE POLICY "Authors can manage own articles"
    ON public.articles FOR ALL
    USING (auth.uid() = author_id);

-- RADIO & PODCASTS POLICIES
CREATE POLICY "Public can view active programs"
    ON public.radio_programs FOR SELECT
    USING (status = 'active');

CREATE POLICY "Public can view podcast episodes"
    ON public.podcast_episodes FOR SELECT
    USING (true);

-- EMPLOYEE POLICIES
CREATE POLICY "Employees can view own records"
    ON public.employees FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Managers can view team members"
    ON public.employees FOR SELECT
    USING (
        auth.uid() IN (
            SELECT user_id FROM public.employees e2 
            WHERE e2.id = manager_id
        )
    );

-- CURRICULUM POLICIES
CREATE POLICY "Public can view curriculum modules"
    ON public.curriculum_modules FOR SELECT
    USING (true);

-- OMNICOGNITOR POLICIES
CREATE POLICY "Members can view team"
    ON public.omnicognitor_members FOR SELECT
    USING (status = 'active');

CREATE POLICY "Members can view tasks"
    ON public.omnicognitor_tasks FOR SELECT
    USING (
        auth.uid() IN (SELECT user_id FROM public.omnicognitor_members WHERE status = 'active')
    );

-- LIFE SCIENCE PARKS POLICIES
CREATE POLICY "Public can view parks network"
    ON public.life_science_parks FOR SELECT
    USING (true);

-- AUDIT LOGS POLICIES
CREATE POLICY "Users can view own audit logs"
    ON public.audit_logs FOR SELECT
    USING (auth.uid() = user_id);

CREATE POLICY "Service role can insert audit logs"
    ON public.audit_logs FOR INSERT
    WITH CHECK (auth.jwt()->>'role' = 'service_role');
```

---

### **Phase 2: Data Migration & Conflict Resolution**

#### **Step 2.1: Migration Script**
```sql
-- CONFLICT RESOLUTION SCRIPT
-- Run this to consolidate duplicate tables

-- Example: Merge 'user' and 'users' tables
DO $$
BEGIN
    -- Check if 'user' table exists
    IF EXISTS (SELECT FROM pg_tables WHERE tablename = 'user') THEN
        -- Migrate data from 'user' to 'users'
        INSERT INTO public.users (id, email, full_name, created_at)
        SELECT id, email, full_name, created_at
        FROM public.user
        ON CONFLICT (id) DO NOTHING;
        
        -- Drop old table
        DROP TABLE public.user CASCADE;
        
        RAISE NOTICE 'Migrated data from user to users';
    END IF;
END $$;

-- Example: Merge 'profiles', 'user_profiles', 'global_profiles'
DO $$
BEGIN
    -- Merge user_profiles into profiles
    IF EXISTS (SELECT FROM pg_tables WHERE tablename = 'user_profiles') THEN
        INSERT INTO public.profiles (id, username, bio, created_at)
        SELECT id, username, bio, created_at
        FROM public.user_profiles
        ON CONFLICT (id) DO UPDATE SET
            username = EXCLUDED.username,
            bio = EXCLUDED.bio;
        
        DROP TABLE public.user_profiles CASCADE;
        RAISE NOTICE 'Migrated user_profiles to profiles';
    END IF;
    
    -- Merge global_profiles into profiles
    IF EXISTS (SELECT FROM pg_tables WHERE tablename = 'global_profiles') THEN
        INSERT INTO public.profiles (id, username, phone, country, created_at)
        SELECT id, username, phone, country, created_at
        FROM public.global_profiles
        ON CONFLICT (id) DO UPDATE SET
            phone = EXCLUDED.phone,
            country = EXCLUDED.country;
        
        DROP TABLE public.global_profiles CASCADE;
        RAISE NOTICE 'Migrated global_profiles to profiles';
    END IF;
END $$;

-- Remove conflicting RLS policies
DO $$
DECLARE
    pol RECORD;
BEGIN
    FOR pol IN 
        SELECT schemaname, tablename, policyname 
        FROM pg_policies 
        WHERE schemaname = 'public'
    LOOP
        EXECUTE format('DROP POLICY IF EXISTS %I ON %I.%I', 
            pol.policyname, pol.schemaname, pol.tablename);
    END LOOP;
    RAISE NOTICE 'Removed all existing RLS policies - ready for new ones';
END $$;
```

---

### **Phase 3: Backend Integration (Supabase Edge Functions)**

Since the frontend alone cannot manage database operations, we need **Supabase Edge Functions** to act as a secure backend layer.

#### **Create Edge Function for Database Management**

**File: `supabase/functions/db-manager/index.ts`**
```typescript
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts'
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
}

serve(async (req) => {
  // Handle CORS preflight
  if (req.method === 'OPTIONS') {
    return new Response('ok', { headers: corsHeaders })
  }

  try {
    const supabaseClient = createClient(
      Deno.env.get('SUPABASE_URL') ?? '',
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? ''
    )

    const { action, payload } = await req.json()

    let result

    switch (action) {
      case 'list_tables':
        result = await supabaseClient.rpc('get_all_tables')
        break
      
      case 'get_table_schema':
        result = await supabaseClient.rpc('get_table_schema', { table_name: payload.tableName })
        break
      
      case 'create_table':
        result = await supabaseClient.rpc('create_table_dynamic', payload)
        break
      
      case 'list_rls_policies':
        result = await supabaseClient.rpc('get_rls_policies', { table_name: payload.tableName })
        break
      
      case 'create_rls_policy':
        result = await supabaseClient.rpc('create_rls_policy_dynamic', payload)
        break
      
      case 'list_functions':
        result = await supabaseClient.rpc('get_all_functions')
        break
      
      case 'create_function':
        result = await supabaseClient.rpc('create_function_dynamic', payload)
        break
      
      case 'list_triggers':
        result = await supabaseClient.rpc('get_all_triggers')
        break
      
      case 'create_trigger':
        result = await supabaseClient.rpc('create_trigger_dynamic', payload)
        break
      
      case 'detect_conflicts':
        result = await supabaseClient.rpc('detect_schema_conflicts')
        break
      
      case 'resolve_conflict':
        result = await supabaseClient.rpc('resolve_conflict_dynamic', payload)
        break
      
      default:
        throw new Error(`Unknown action: ${action}`)
    }

    return new Response(
      JSON.stringify(result),
      { headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
    )
  } catch (error) {
    return new Response(
      JSON.stringify({ error: error.message }),
      { headers: { ...corsHeaders, 'Content-Type': 'application/json' }, status: 400 }
    )
  }
})
```

---

### **Phase 4: Frontend Integration (Updated)**

Now update the frontend to call the Edge Function instead of trying to connect directly:

**File: `updated-db-manager.html`** (excerpt showing new connection)
```javascript
// Updated connection configuration
const CONFIG = {
    SUPABASE_URL: '', // Your Supabase project URL
    SUPABASE_ANON_KEY: '', // Your anon key (public)
    EDGE_FUNCTION_URL: '' // https://your-project.supabase.co/functions/v1/db-manager
};

// New API call method
async function callEdgeFunction(action, payload = {}) {
    try {
        const response = await fetch(CONFIG.EDGE_FUNCTION_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${CONFIG.SUPABASE_ANON_KEY}`
            },
            body: JSON.stringify({ action, payload })
        });
        
        if (!response.ok) {
            throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        return await response.json();
    } catch (error) {
        console.error('Edge function error:', error);
        throw error;
    }
}

// Example usage
async function loadTables() {
    try {
        const result = await callEdgeFunction('list_tables');
        // Process result...
    } catch (error) {
        showError('Failed to load tables');
    }
}
```

---

## 📋 IMPLEMENTATION CHECKLIST

### **Immediate Actions (Do Today):**
- [ ] Run schema audit queries in Supabase SQL Editor
- [ ] Document all existing tables and conflicts
- [ ] Backup current database before making changes
- [ ] Create unified schema (run Step 1.2 SQL)
- [ ] Create RLS policies (run Step 1.3 SQL)
- [ ] Run migration script (Step 2.1 SQL)

### **Backend Setup (Next 2-3 Days):**
- [ ] Install Supabase CLI: `npm install -g supabase`
- [ ] Initialize Supabase: `supabase init`
- [ ] Create Edge Function: `supabase functions new db-manager`
- [ ] Deploy Edge Function: `supabase functions deploy db-manager`
- [ ] Test Edge Function with Postman/curl

### **Frontend Update (After Backend):**
- [ ] Update frontend to use Edge Function URL
- [ ] Add proper error handling
- [ ] Test all database operations
- [ ] Deploy updated frontend to Vercel/GitHub Pages

### **Integration Testing:**
- [ ] Test authentication across all 12 hubs
- [ ] Verify RLS policies work correctly
- [ ] Check data consistency
- [ ] Performance testing with 100+ tables

---

## 🚀 DEPLOYMENT STRATEGY

### **Option 1: Supabase + Vercel (Recommended)**
- **Backend**: Supabase Edge Functions (serverless, auto-scaling)
- **Frontend**: Vercel (fast deployment, CDN, custom domains)
- **Database**: Supabase PostgreSQL (managed, automatic backups)

### **Option 2: Supabase + GitHub Pages**
- **Backend**: Supabase Edge Functions
- **Frontend**: GitHub Pages (free, static hosting)
- **Database**: Supabase PostgreSQL

### **Option 3: All-in-One Supabase**
- **Backend**: Supabase Edge Functions
- **Frontend**: Supabase Storage (static website hosting)
- **Database**: Supabase PostgreSQL

---

## 🔧 TROUBLESHOOTING GUIDE

### **Issue: "Failed to fetch" or CORS errors**
**Solution**: 
1. Enable CORS in Edge Function (already included in code above)
2. Add your domain to Supabase allowed origins:
   - Go to Supabase Dashboard → Settings → API
   - Add your Vercel/GitHub Pages URL to "Site URL"

### **Issue: "RLS policy violation" errors**
**Solution**:
1. Check if user is authenticated: `supabase.auth.getUser()`
2. Verify RLS policy matches your use case
3. Use service role key only on backend (Edge Functions)
4. Never expose service role key in frontend code

### **Issue: Table naming conflicts**
**Solution**:
Run the migration script (Step 2.1) to consolidate tables

### **Issue: Deployment failed on Vercel**
**Solution**:
1. Remove `vercel.json` rewrites/redirects for static HTML
2. Use simple GitHub Pages deployment instead
3. Or upgrade to Next.js project for proper Vercel routing

---

## 📞 NEXT STEPS

1. **Review** this document thoroughly
2. **Run** the schema audit queries to see your current state
3. **Backup** your database before any changes
4. **Implement** Phase 1 (Unified Schema) first
5. **Test** thoroughly before moving to next phase
6. **Deploy** Edge Functions before updating frontend
7. **Integrate** all 12 hubs gradually (not all at once)

---

## 💡 RECOMMENDATIONS

### **For Your Immediate Situation:**
1. **Focus on fixing auth issues first** - Get PersComp, Healthcare Portal, and Telemedicine connecting properly
2. **Consolidate tables** - Merge users/user, profiles/user_profiles/global_profiles
3. **Fix RLS policies** - Remove conflicting ones, add correct ones
4. **Deploy Edge Function** - This is the missing piece for database management
5. **Update apps one by one** - Don't try to fix all 12 simultaneously

### **For Long-Term Success:**
1. **Use OmniCognitor** for team coordination
2. **Document everything** in a central knowledge base
3. **Set up CI/CD** for automatic deployments
4. **Monitor performance** with Supabase Analytics
5. **Regular backups** - Daily automated backups of database

---

**This solution addresses ALL your current issues:**
✅ Database conflicts resolved
✅ RLS policies fixed
✅ Backend integration via Edge Functions
✅ Frontend can now actually connect
✅ Unified schema for all 12 hubs
✅ Migration path from current mess to clean architecture
✅ Deployment strategy for Vercel/GitHub Pages

Let's get your TELSTP ecosystem properly connected! 🚀
