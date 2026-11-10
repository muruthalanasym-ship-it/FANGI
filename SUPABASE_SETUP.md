# FANGI public gallery uploads — one-time setup

The website is hosted as a static GitHub Pages site. A static page cannot securely store admin passwords or accept permanent public uploads by itself. This implementation uses Supabase Auth + Storage. Do not put a Supabase service_role key in the website.

## 1. Create the storage project
1. Create a project at https://supabase.com/dashboard.
2. In Project Settings → API, copy the Project URL and the anon/public key.
3. Edit `assets/supabase-config.js` in this GitHub repository and replace the two placeholder values. The anon/public key is intended for browser use; never use the service_role key.

## 2. Create the public image bucket
In Supabase Storage, create a bucket named `fangi-gallery` and set it to **Public**. Set the file size limit to 8 MB and allowed MIME types to `image/jpeg`, `image/png`, and `image/webp`.

## 3. Lock down uploads to the owner
In Supabase Authentication → Providers → Email, keep email/password sign-in enabled and disable public sign-ups. Then create your own admin user from Authentication → Users → Add user. Use an email and password only you control.

Open the SQL Editor and run:

```sql
create policy "Public can view FANGI gallery photos"
on storage.objects for select
to anon, authenticated
using (bucket_id = 'fangi-gallery');

create policy "Signed-in admins can upload FANGI gallery photos"
on storage.objects for insert
to authenticated
with check (bucket_id = 'fangi-gallery');

create policy "Signed-in admins can delete FANGI gallery photos"
on storage.objects for delete
to authenticated
using (bucket_id = 'fangi-gallery');
```

Because public sign-ups are disabled, only the user account you create can sign in through the website admin page. Do not create additional accounts unless you want to authorize another uploader.

## 4. Use the upload page
After GitHub Pages publishes the changes, open:
https://muruthalanasym-ship-it.github.io/FANGI/admin.html

Sign in with the admin email/password you created, select JPG/PNG/WebP service photos (8 MB maximum each), and upload. Photos are then listed in the public FANGI Gallery. You can delete photos from the same admin page.

## Notes
- Existing sample gallery cards remain in place; uploaded project photos are added to the top of the gallery.
- If you later want to change the admin password, do so in Supabase Authentication.
- The bucket is public for reading images, but upload and delete operations require an authenticated user.
