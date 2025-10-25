# TIFF-images-Convert-Into-JPEG-or-Others-format

# Python Environment
    python --version
    
    # If you don’t have it:
    🔗 https://www.python.org/downloads/

# Required Python Packages

    pip install cx_Oracle Pillow


# Write Python Code

    import cx_Oracle
    from PIL import Image
    import io
    
    # === Step 1: Connect to Oracle ===
    try:
        conn = cx_Oracle.connect("username/password@//hostname:port/service_name")
        cursor = conn.cursor()
        print("✅ Connected to Oracle.")
    except cx_Oracle.DatabaseError as e:
        print("❌ Connection failed:", e)
        exit()
    
    # === Step 2: Fetch TIFF image where JPEG is not yet stored
    try:
        cursor.execute("""
            SELECT user_id, signature
            FROM user_profiles
            WHERE signature IS NOT NULL
        """)
    
        rows = cursor.fetchall()
        print(f"🔍 Found {len(rows)} matching rows.")
    
    except cx_Oracle.DatabaseError as e:
        print("❌ Failed to fetch rows:", e)
        cursor.close()
        conn.close()
        exit()
    
    # === Step 3: Convert TIFF to JPEG and update database
    for user_id, tiff_blob in rows:
        try:
            tiff_bytes = tiff_blob.read()
            tiff_stream = io.BytesIO(tiff_bytes)
    
            image = Image.open(tiff_stream)
    
            # ✅ Check if the image is TIFF
            if image.format != 'TIFF':
                print(f"⚠️ USER_ID {user_id}: Skipped (not a TIFF image, found {image.format})")
                continue  # Skip this one
    
            # ✅ Convert to RGB if necessary
            if image.mode != "RGB":
                image = image.convert("RGB")
    
            # ✅ Convert to JPEG
            jpeg_stream = io.BytesIO()
            image.save(jpeg_stream, format="JPEG")
            jpeg_data = jpeg_stream.getvalue()
    
            # ✅ Update the database
            cursor.execute("""
                UPDATE user_profiles
                SET photo = :blobdata
                WHERE user_id = :id
            """, {
                "blobdata": jpeg_data,
                "id": user_id
            })
    
            if cursor.rowcount > 0:
                print(f"✔ USER_ID {user_id} updated successfully.")
            else:
                print(f"⚠️ USER_ID {user_id} not updated — no matching row.")
    
        except Exception as e:
            print(f"❌ Failed for USER_ID {user_id}: {e}")
    
    # === Step 4: Commit and close
    try:
        conn.commit()
        print("✅ Changes committed.")
    except Exception as e:
        print(f"❌ Commit failed: {e}")
    
    cursor.close()
    conn.close()
    print("✅ Connection closed.")
