# Supabase Row Level Security (RLS) Policies

## ⚠️ CRITICAL: These policies MUST be implemented in your Supabase dashboard

### Prerequisites
1. Go to your Supabase project dashboard
2. Navigate to **Authentication** → **Policies**
3. Select the `files` table
4. Enable Row Level Security if not already enabled

---

## Step 1: Enable RLS on Files Table

```sql
-- Enable Row Level Security
ALTER TABLE files ENABLE ROW LEVEL SECURITY;
```

---

## Step 2: Create SELECT Policy (Read Access)

**Policy Name:** `Users can view own files`

```sql
CREATE POLICY "Users can view own files"
ON files
FOR SELECT
USING (auth.uid() = user_id);
```

**What this does:**
- Users can only SELECT/read files where their `auth.uid()` matches the `user_id` column
- Prevents users from querying other users' files

---

## Step 3: Create INSERT Policy (Upload Access)

**Policy Name:** `Users can insert own files`

```sql
CREATE POLICY "Users can insert own files"
ON files
FOR INSERT
WITH CHECK (auth.uid() = user_id);
```

**What this does:**
- Users can only INSERT files with their own `user_id`
- Prevents users from creating files under another user's ID

---

## Step 4: Create DELETE Policy (Delete Access)

**Policy Name:** `Users can delete own files`

```sql
CREATE POLICY "Users can delete own files"
ON files
FOR DELETE
USING (auth.uid() = user_id);
```

**What this does:**
- Users can only DELETE files they own
- Prevents users from deleting other users' files

---

## Step 5: Create UPDATE Policy (Modify Access)

**Policy Name:** `Users can update own files`

```sql
CREATE POLICY "Users can update own files"
ON files
FOR UPDATE
USING (auth.uid() = user_id)
WITH CHECK (auth.uid() = user_id);
```

**What this does:**
- Users can only UPDATE files they own
- Prevents modification of other users' files

---

## Storage Bucket Policies

### Enable RLS on Storage

```sql
-- Storage policy: Users can only access their own files
CREATE POLICY "Users can access own files"
ON storage.objects
FOR SELECT
USING (
  bucket_id = 'files' AND 
  (storage.foldername(name))[1] = auth.uid()::text
);

CREATE POLICY "Users can upload to own folder"
ON storage.objects
FOR INSERT
WITH CHECK (
  bucket_id = 'files' AND 
  (storage.foldername(name))[1] = auth.uid()::text
);

CREATE POLICY "Users can delete own files"
ON storage.objects
FOR DELETE
USING (
  bucket_id = 'files' AND 
  (storage.foldername(name))[1] = auth.uid()::text
);
```

---

## Storage Bucket Configuration

### Go to Storage → files bucket → Configuration

1. **Make bucket private:**
   - Uncheck "Public bucket"

2. **Set file size limit:**
   - Maximum file size: `52428800` (50MB)

3. **Set allowed MIME types:**
```json
[
  "text/plain",
  "text/x-python",
  "application/javascript",
  "application/typescript",
  "application/json",
  "application/zip",
  "application/pdf"
]
```

---

## Verification

### Test RLS Policies

Run these queries in the Supabase SQL Editor to verify:

```sql
-- Test 1: Check if RLS is enabled
SELECT tablename, rowsecurity 
FROM pg_tables 
WHERE tablename = 'files';
-- Should return: rowsecurity = true

-- Test 2: List all policies
SELECT * FROM pg_policies 
WHERE tablename = 'files';
-- Should show 4 policies (SELECT, INSERT, UPDATE, DELETE)

-- Test 3: Try to access files (as authenticated user)
SELECT * FROM files;
-- Should only return files where user_id = your auth.uid()
```

---

## Common Issues

### Issue: "new row violates row-level security policy"
**Solution:** Make sure the `user_id` being inserted matches `auth.uid()`

### Issue: "permission denied for table files"
**Solution:** RLS is enabled but no policies exist. Create the policies above.

### Issue: Users can still see other users' files
**Solution:** 
1. Verify RLS is enabled: `ALTER TABLE files ENABLE ROW LEVEL SECURITY;`
2. Check policies exist and are correct
3. Ensure client-side code uses authenticated Supabase client

---

## Security Best Practices

1. ✅ **Always use RLS** - Never rely on client-side filtering alone
2. ✅ **Test policies** - Use Supabase's policy testing feature
3. ✅ **Audit regularly** - Review policies monthly
4. ✅ **Use auth.uid()** - Always reference authenticated user ID
5. ✅ **Private buckets** - Never make storage buckets public unless necessary

---

## Implementation Checklist

- [ ] Enable RLS on `files` table
- [ ] Create SELECT policy
- [ ] Create INSERT policy  
- [ ] Create DELETE policy
- [ ] Create UPDATE policy
- [ ] Enable storage RLS policies
- [ ] Make storage bucket private
- [ ] Set file size limits
- [ ] Set allowed MIME types
- [ ] Test with multiple users
- [ ] Verify unauthorized access is blocked

---

**IMPORTANT:** After implementing these policies, the IDOR vulnerability will be fixed. Users will only be able to access their own files, even if they know the file paths of other users.
