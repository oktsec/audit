# CONDITIONAL: File Uploads

Only load if upload handling detected (multer, formidable, busboy, express-fileupload, python-multipart, uploadthing).

| Check | What to look for | Issue |
|-------|-----------------|-------|
| File size limit | Search multer/formidable/busboy config for `limits`, `maxFileSize`, `fileSizeLimit`. If no limit set: HIGH | Attacker uploads 10GB file, fills your disk |
| File type validation | Search for `mimetype`, `fileFilter`, `content_type`, `allowed_extensions`. If no type check: HIGH | Attacker uploads executable, PHP shell, or HTML file (stored XSS) |
| Filename sanitization | Search for `path.basename`, `sanitize`, `originalname` used directly in `path.join`. If original filename used as-is in file path: HIGH | Path traversal via filename (`../../etc/passwd`) |
| Storage location | Check if uploads go to a publicly accessible directory (`public/`, `static/`, `uploads/` served by web server) without access control | Direct URL access to any uploaded file |
