/* 
================================================================================
=                          COMBINED ANDROID PROJECT CODE                       =
=    (AndroidManifest.xml, activity_main.xml, build.gradle, DatabaseHelper.kt, =
=                  and MainActivity.kt in ONE snippet)                         =
================================================================================

--------------------------------------------------------------------------------
SECTION A: AndroidManifest.xml
--------------------------------------------------------------------------------
*/
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.ariatheeditor">

    <!-- INTERNET permission is required for PostgreSQL database connections -->
    <uses-permission android:name="android.permission.INTERNET" />

    <!-- 
         For Android 13+ (API level 33+):
         - READ_MEDIA_IMAGES for images, READ_MEDIA_AUDIO for audio 
         For older versions, see fallback in MainActivity 
    -->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
    <uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.App">
        
        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>
</manifest>


/*
--------------------------------------------------------------------------------
SECTION B: activity_main.xml
--------------------------------------------------------------------------------

Typically in: app/src/main/res/layout/activity_main.xml
*/
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android">

    <FrameLayout
        android:id="@+id/root_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!-- PhotoEditorView from ja.burhanrashid52.photoeditor library -->
        <ja.burhanrashid52.photoeditor.PhotoEditorView
            android:id="@+id/photoEditorView"
            android:layout_width="match_parent"
            android:layout_height="match_parent" />

    </FrameLayout>
</layout>


/*
--------------------------------------------------------------------------------
SECTION C: build.gradle (Module-level)
--------------------------------------------------------------------------------

Typically in: app/build.gradle
*/
plugins {
    id("com.android.application")
    id("kotlin-android")
    // For annotation processing (if needed for other libraries)
    id("kotlin-kapt")
}

android {
    namespace = "com.example.ariatheeditor"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.ariatheeditor"
        minSdk = 21
        targetSdk = 34

        versionCode = 1
        versionName = "1.0"

        multiDexEnabled = true
    }

    buildFeatures {
        // Enable Jetpack Compose
        compose = true
    }

    composeOptions {
        kotlinCompilerExtensionVersion = "1.4.7"
    }

    packagingOptions {
        resources {
            excludes += "/META-INF/{AL2.0,LGPL2.1}"
        }
    }
}

dependencies {
    // AndroidX Core Libraries
    implementation("androidx.core:core-ktx:1.10.1")
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.google.android.material:material:1.9.0")

    // Jetpack Compose
    implementation("androidx.compose.ui:ui:1.4.3")
    implementation("androidx.compose.material3:material3:1.1.0")
    implementation("androidx.compose.ui:ui-tooling-preview:1.4.3")
    debugImplementation("androidx.compose.ui:ui-tooling:1.4.3")

    // PhotoEditor library
    implementation("ja.burhanrashid52:photoeditor:2.0.0")

    // Coil for image loading in Compose
    implementation("io.coil-kt:coil-compose:2.2.2")

    // Coroutines
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.0")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.0")

    // PostgreSQL JDBC Driver (Demo only; not recommended directly on Android)
    implementation("org.postgresql:postgresql:42.6.0")
}


/*
--------------------------------------------------------------------------------
SECTION D: DatabaseHelper.kt
--------------------------------------------------------------------------------

Typically in: app/src/main/java/com/example/ariatheeditor/DatabaseHelper.kt
*/
package com.example.ariatheeditor

import java.sql.Connection
import java.sql.DriverManager
import java.sql.SQLException

object DatabaseHelper {

    // Avoid hardcoding credentials in production.
    // Use secure methods (environment variables, server auth) in real apps.
    private const val JDBC_URL = "jdbc:postgresql://<YOUR_DB_HOST>:5432/<YOUR_DB_NAME>"
    private const val USER = "<YOUR_DB_USER>"
    private const val PASSWORD = "<YOUR_DB_PASSWORD>"

    fun connect(): Connection? {
        return try {
            Class.forName("org.postgresql.Driver")
            DriverManager.getConnection(JDBC_URL, USER, PASSWORD).also {
                println("Database connected successfully!")
            }
        } catch (e: ClassNotFoundException) {
            e.printStackTrace()
            null
        } catch (e: SQLException) {
            e.printStackTrace()
            null
        }
    }
}


/*
--------------------------------------------------------------------------------
SECTION E: MainActivity.kt
--------------------------------------------------------------------------------

Typically in: app/src/main/java/com/example/ariatheeditor/MainActivity.kt
*/
package com.example.ariatheeditor

import android.Manifest
import android.app.AlertDialog
import android.content.Context
import android.content.SharedPreferences
import android.content.pm.PackageManager
import android.media.MediaPlayer
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.compose.foundation.Image
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Text
import androidx.compose.material3.lightColorScheme
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.unit.dp
import androidx.core.app.ActivityCompat
import androidx.core.content.ContextCompat
import coil.compose.rememberAsyncImagePainter
import ja.burhanrashid52.photoeditor.PhotoEditor
import ja.burhanrashid52.photoeditor.PhotoEditorView
import ja.burhanrashid52.photoeditor.PhotoFilter
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.GlobalScope
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {

    private lateinit var sharedPreferences: SharedPreferences

    private var selectedImageUri by mutableStateOf<Uri?>(null)
    private var mediaPlayer: MediaPlayer? = null
    private lateinit var photoEditor: PhotoEditor

    // Activity Result Launcher for selecting an image
    private val pickImage =
        registerForActivityResult(ActivityResultContracts.GetContent()) { uri: Uri? ->
            uri?.let { selectedImageUri = it }
        }

    // Activity Result Launcher for selecting audio
    private val pickAudio =
        registerForActivityResult(ActivityResultContracts.GetContent()) { uri: Uri? ->
            uri?.let { playAudio(it) }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Initialize SharedPreferences
        sharedPreferences = getSharedPreferences("app_prefs", Context.MODE_PRIVATE)

        // Check or request permissions
        checkRequiredPermissions()

        // Show a consent dialog if it's the first time the user launches the app
        if (isFirstLaunch()) {
            showConsentDialog()
        } else {
            startApp()
        }
    }

    private fun checkRequiredPermissions() {
        val permissionsNeeded = mutableListOf<String>()

        // For Android 13+ use READ_MEDIA_* instead of READ_EXTERNAL_STORAGE
        if (Build.VERSION.SDK_INT >= 33) {
            permissionsNeeded.add(Manifest.permission.READ_MEDIA_IMAGES)
            permissionsNeeded.add(Manifest.permission.READ_MEDIA_AUDIO)
        } else {
            permissionsNeeded.add(Manifest.permission.READ_EXTERNAL_STORAGE)
        }

        val missingPermissions = permissionsNeeded.filter {
            ContextCompat.checkSelfPermission(this, it) != PackageManager.PERMISSION_GRANTED
        }

        if (missingPermissions.isNotEmpty()) {
            ActivityCompat.requestPermissions(this, missingPermissions.toTypedArray(), 1001)
        }
    }

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == 1001) {
            val allGranted = grantResults.all { it == PackageManager.PERMISSION_GRANTED }
            if (!allGranted) {
                Toast.makeText(
                    this,
                    "Some permissions are denied. Certain features may not work.",
                    Toast.LENGTH_LONG
                ).show()
            }
        }
    }

    private fun isFirstLaunch(): Boolean {
        return sharedPreferences.getBoolean("is_first_launch", true)
    }

    private fun showConsentDialog() {
        AlertDialog.Builder(this).apply {
            setTitle("Terms & Conditions")
            setMessage("Please accept our Terms, Privacy Policy, and confirm you are 18+ to proceed.")
            setCancelable(false)
            setPositiveButton("Accept") { _, _ ->
                sharedPreferences.edit().putBoolean("is_first_launch", false).apply()
                startApp()
            }
            setNegativeButton("Decline") { _, _ ->
                finish()
            }
        }.show()
    }

    private fun startApp() {
        // Initialize UI with Jetpack Compose
        setContent {
            MaterialTheme(colorScheme = lightColorScheme()) {
                EditorScreen()
            }
        }

        // Optionally, you can use the XML layout:
        // setContentView(R.layout.activity_main)
        // val photoEditorView = findViewById<PhotoEditorView>(R.id.photoEditorView)

        // Demonstration: create PhotoEditorView programmatically
        val photoEditorView = PhotoEditorView(this).apply {
            id = R.id.photoEditorView
        }

        // Setup PhotoEditor
        photoEditor = PhotoEditor.Builder(this, photoEditorView)
            .setPinchTextScalable(true) // Allow pinch-to-scale for text
            .build()

        // Initiate database connection (DEMO ONLY)
        // Consider using lifecycleScope or other structured concurrency
        GlobalScope.launch(Dispatchers.IO) {
            DatabaseHelper.connect()
        }
    }

    @Composable
    fun EditorScreen() {
        val context = LocalContext.current

        Column(
            modifier = Modifier
                .fillMaxSize()
                .background(Color(0xFFF5F5F5)),
            verticalArrangement = Arrangement.Center,
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            // Display selected image or a placeholder
            selectedImageUri?.let { uri ->
                Image(
                    painter = rememberAsyncImagePainter(uri),
                    contentDescription = "Selected Image",
                    modifier = Modifier
                        .size(250.dp)
                        .clickable {
                            pickImage.launch("image/*")
                        }
                )
            } ?: Box(
                modifier = Modifier
                    .size(250.dp)
                    .background(Color.Gray, shape = RoundedCornerShape(12.dp))
                    .clickable { pickImage.launch("image/*") },
                contentAlignment = Alignment.Center
            ) {
                Text("Tap to Upload Image", color = Color.White)
            }

            Spacer(modifier = Modifier.height(16.dp))

            Button(onClick = { pickImage.launch("image/*") }) {
                Text("Upload Image")
            }

            Spacer(modifier = Modifier.height(8.dp))

            Button(onClick = { addEmoji() }) {
                Text("Add Emoji")
            }

            Spacer(modifier = Modifier.height(8.dp))

            Button(onClick = { applyFilter(PhotoFilter.BLACK_WHITE) }) {
                Text("Apply B&W Filter")
            }

            Spacer(modifier = Modifier.height(8.dp))

            Button(onClick = { pickAudio.launch("audio/*") }) {
                Text("Select Audio")
            }
        }
    }

    private fun addEmoji() {
        if (::photoEditor.isInitialized) {
            photoEditor.addEmoji("\uD83D\uDE0A") // Smiley emoji
            Toast.makeText(this, "Emoji added!", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(this, "PhotoEditor not initialized.", Toast.LENGTH_SHORT).show()
        }
    }

    private fun applyFilter(filter: PhotoFilter) {
        if (::photoEditor.isInitialized) {
            photoEditor.setFilterEffect(filter)
            Toast.makeText(this, "Filter applied: $filter", Toast.LENGTH_SHORT).show()
        }
    }

    private fun playAudio(uri: Uri) {
        mediaPlayer?.release()
        mediaPlayer = MediaPlayer.create(this, uri)
        mediaPlayer?.start()
        Toast.makeText(this, "Playing audio...", Toast.LENGTH_SHORT).show()
    }
}

/* 
================================================================================
END OF COMBINED CODE
================================================================================