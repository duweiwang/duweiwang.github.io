---
title: Android存储
date: 2026-01-13 21:25:59
tags:
categories:
description:
---


#### 一、Android存储路径相关API
| API                                   | 示例路径                                                  | 分类                       | 说明                                 |
|---------------------------------------|----------------------------------------------------------|----------------------------|------------------------------------|
| context.getFilesDir()                 | /data/data/com.example.app/files                         | Internal Storage           | 应用私有，无权限需求，卸载即删除                   |
| context.getCacheDir()                 | /data/data/com.example.app/cache                         | Internal Storage Cache     | 系统可能清理，用于临时文件                      |
| context.getExternalFilesDir(null)     | /storage/emulated/0/Android/data/com.example.app/files   | External Private Storage   | App专属，无需权限（Android 4.4+）           |
| context.getExternalFilesDir(type)     | /storage/emulated/0/Android/data/com.example.app/Pictures | External Private Storage   | 媒体分类目录（如 Pictures/Music），无需权限      |
| Environment.getExternalStorageDirectory()     | /storage/emulated/0 | External Public Storage   | 外部根目录                              |
| Environment.getExternalStoragePublicDirectory(DOWNLOADS) | /storage/emulated/0/Downloads          | External Public Storage    | 所有App可访问，Android 10+受限制            |
| Environment.getExternalStoragePublicDirectory(PICTURES) | /storage/emulated/0/Pictures            | External Public Storage    | Android 10+需 MediaStore/SAF，直接访问受限 |


#### 二、MediaStore API 和 Storage Access Framework (SAF)
##### 2.1 MediaStore API：系统管理的媒体库
适用场景：访问图片、视频、音频等媒体文件

```kotlin
// 查询所有图片
fun queryImages(context: Context): List<ImageItem> {
    val images = mutableListOf<ImageItem>()
    val projection = arrayOf(
        MediaStore.Images.Media._ID,
        MediaStore.Images.Media.DISPLAY_NAME,
        MediaStore.Images.Media.DATE_TAKEN,
        MediaStore.Images.Media.SIZE)

    val sortOrder = "${MediaStore.Images.Media.DATE_TAKEN} DESC"
    
    context.contentResolver.query(
        MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
        projection,
        null,
        null,
        sortOrder
    )?.use { cursor ->
        val idColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media._ID)
        val nameColumn = cursor.getColumnIndexOrThrow(MediaStore.Images.Media.DISPLAY_NAME)
        
        while (cursor.moveToNext()) {
            val id = cursor.getLong(idColumn)
            val name = cursor.getString(nameColumn)
            val contentUri = ContentUris.withAppendedId(
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI, id)
            
            images.add(ImageItem(id, name, contentUri))
        }
    }
    return images
}

// 保存图片到媒体库
fun saveImageToMediaStore(context: Context, bitmap: Bitmap): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Images.Media.DISPLAY_NAME, "photo_${System.currentTimeMillis()}.jpg")
        put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            put(MediaStore.Images.Media.RELATIVE_PATH, Environment.DIRECTORY_PICTURES + "/MyApp")
            put(MediaStore.Images.Media.IS_PENDING, 1)
        }
    }

    val resolver = context.contentResolver
    val uri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, contentValues)
    
    uri?.let {
        resolver.openOutputStream(it)?.use { outputStream ->
            bitmap.compress(Bitmap.CompressFormat.JPEG, 90, outputStream)
        }
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
            contentValues.clear()
            contentValues.put(MediaStore.Images.Media.IS_PENDING, 0)
            resolver.update(uri, contentValues, null, null)
        }
    }
    
    return uri
}
```


##### 2.2 Storage Access Framework (SAF)：用户控制的文件访问
适用场景：访问任意类型文件，让用户完全控制

```kotlin
// 打开文档
fun openDocument(activity: Activity) {
    val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
        addCategory(Intent.CATEGORY_OPENABLE)
        type = "*/*"
        putExtra(Intent.EXTRA_MIME_TYPES, arrayOf(
            "image/*",
            "application/pdf",
            "text/plain"
        ))
    }
    activity.startActivityForResult(intent, REQUEST_OPEN_DOCUMENT)
}

// 创建文档
fun createDocument(activity: Activity, mimeType: String, filename: String) {
    val intent = Intent(Intent.ACTION_CREATE_DOCUMENT).apply {
        addCategory(Intent.CATEGORY_OPENABLE)
        type = mimeType
        putExtra(Intent.EXTRA_TITLE, filename)
    }
    activity.startActivityForResult(intent, REQUEST_CREATE_DOCUMENT)
}

// 处理返回结果
override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
    super.onActivityResult(requestCode, resultCode, data)

    if (resultCode == Activity.RESULT_OK && data != null) {
        val uri = data.data
        when (requestCode) {
            REQUEST_OPEN_DOCUMENT -> handleOpenDocument(uri)
            REQUEST_CREATE_DOCUMENT -> handleCreateDocument(uri)
        }
        
        // 获取持久化权限（Android 11+）
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            uri?.let {
                contentResolver.takePersistableUriPermission(
                    it,
                    Intent.FLAG_GRANT_READ_URI_PERMISSION or
                    Intent.FLAG_GRANT_WRITE_URI_PERMISSION
                )
            }
        }
    }
}

// 使用DocumentFile操作文件
fun processDocumentFile(context: Context, uri: Uri) {
    val documentFile = DocumentFile.fromSingleUri(context, uri)
    documentFile?.let {
        val fileName = it.name
        val fileSize = it.length()
        val mimeType = it.type

        // 读取文件内容
        context.contentResolver.openInputStream(uri)?.use { inputStream ->
            // 处理文件流
        }
        
        // 写入文件内容
        context.contentResolver.openOutputStream(uri)?.use { outputStream ->
            // 写入数据
        }
    }
}
```

##### 2.3 对比总结

| 特性             | MediaStore API                     | Storage Access Framework (SAF)         |
|------------------|------------------------------------|----------------------------------------|
| 访问范围         | 媒体文件（图片、视频、音频）        | 所有类型文件                           |
| 控制权           | 系统管理                           | 用户控制                               |
| 权限需求         | 运行时权限（Android 13+）          | 无需存储权限                           |
| 数据持久性       | 文件持续存在                       | 需要持久化权限（persistable permission）|
| 使用场景         | 媒体密集型应用                     | 文件管理器、文档编辑器                 |


#### 三、权限请求的版本演进
##### 3.1 Android 5.1 (API 22) 及之前
```xml
<!-- 安装时请求所有权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
##### 3.2 Android 6.0-9.0 (API 23-28)
运行时权限系统引入

```kotlin
// 检查并请求权限
private fun requestStoragePermission() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        when {
            // 已有权限
            ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) 
                    == PackageManager.PERMISSION_GRANTED -> {
                // 执行操作
            }
            // 需要解释权限用途
            shouldShowRequestPermissionRationale(Manifest.permission.READ_EXTERNAL_STORAGE) -> {
                showPermissionRationale()
            }
            // 直接请求权限
            else -> {
                requestPermissions(arrayOf(
                    Manifest.permission.READ_EXTERNAL_STORAGE,
                    Manifest.permission.WRITE_EXTERNAL_STORAGE),
                REQUEST_STORAGE_PERMISSION)
            }
        }
    }
}
```

##### 3.3 Android 10 (API 29)
分区存储引入，WRITE_EXTERNAL_STORAGE权限作用域变化

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<!-- 在Android 10上，WRITE_EXTERNAL_STORAGE只能访问媒体文件 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" 
                 android:maxSdkVersion="28" />
```

一般情况下都会使用：
```xml
 <application
    android:requestLegacyExternalStorage="true"
    />
```
##### 3.4 Android 11 (API 30)
强制分区存储，细化权限分组

```kotlin
// 请求管理所有文件访问权限（MANAGE_EXTERNAL_STORAGE）
private fun requestManageAllFilesPermission() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        if (!Environment.isExternalStorageManager()) {
            val intent = Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION)
            intent.data = Uri.parse("package:${packageName}")
            startActivity(intent)
        }
    }
}
```

##### 3.5 Android 13 (API 33)
读取外部存储的权限被废弃。替换成三个权限：
+ Manifest.permission.READ_MEDIA_IMAGES
+ Manifest.permission.READ_MEDIA_VIDEO
+ Manifest.permission.READ_MEDIA_AUDIO

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
    android:maxSdkVersion="32" />
```

```kotlin
// 检查并请求特定媒体类型权限
private fun requestMediaPermissions() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        val permissionsToRequest = mutableListOf<String>()

        // 根据需求请求特定权限
        if (needPhotoAccess) {
            permissionsToRequest.add(Manifest.permission.READ_MEDIA_IMAGES)
        }
        if (needVideoAccess) {
            permissionsToRequest.add(Manifest.permission.READ_MEDIA_VIDEO)
        }
        if (needAudioAccess) {
            permissionsToRequest.add(Manifest.permission.READ_MEDIA_AUDIO)
        }
        
        if (permissionsToRequest.isNotEmpty()) {
            requestPermissions(
                permissionsToRequest.toTypedArray(),
                REQUEST_MEDIA_PERMISSIONS
            )
        }
    } else {
        // Android 12及以下使用旧权限
        requestPermissions(
            arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE),
            REQUEST_STORAGE_PERMISSION
        )
    }
}
```

当申请Manifest.permission.READ_MEDIA_AUDIO权限时，系统弹窗供用户选择：
+ Allow
+ Deny
当申请Manifest.permission.READ_MEDIA_VIDEO权限时，系统弹窗供用户选择：
+ Allow
+ Deny
+ Limit Access

如果用户选择了Limit Access，接下来用户需要选择一部分资源供App使用。目前没有API可以判断到当前是Limit Access状态。
当前授予Manifest.permission.READ_MEDIA_VIDEO权限时，checkSelfPermission返回true
下次启动时，判断Manifest.permission.READ_MEDIA_VIDEO权限，checkSelfPermission返回false

区分Limit Access状态:
```kotlin

suspend fun isLimitAccess(context: Context): Boolean {
    val noPermission = ContextCompat.checkSelfPermission(mContext, Manifest.permission.READ_MEDIA_VIDEO) ==
            PackageManager.PERMISSION_DENIED
    return noPermission && canReadAnyVideo(context)
}

/**
 * Android 13之后，选择limit access之后，第二次启动就没权限了，但是其实可以访问到用户之前选择的文件。
 */
suspend fun canReadAnyVideo(context: Context): Boolean =
    withContext(Dispatchers.IO) {
        val projection = arrayOf(MediaStore.Video.Media._ID)
        context.contentResolver.query(
            MediaStore.Video.Media.EXTERNAL_CONTENT_URI,
            projection,
            null,
            null,
            null
        )?.use { cursor ->
            cursor.moveToFirst()
        } ?: false
    }
```


#### 四、各存储方案在不同版本的兼容性矩阵
##### 4.1 File API的限制情况
| Android版本      | File API访问范围                | 是否需要权限             | 备注                 |
|------------------|--------------------------------|--------------------------|----------------------|
| ≤ 9 (API 28)     | 整个外部存储                    | 读写都需要权限           | 完全访问             |
| 10 (API 29)      | 应用私有目录 + 媒体文件          | 读写媒体文件需要权限     | 分区存储可选         |
| ≥ 11 (API 30+)   | 仅应用私有目录                   | 访问媒体文件需通过MediaStore | 强制分区存储     |


兼容性处理示例：

```kotlin
fun getAppSpecificFile(context: Context, filename: String): File {
    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
    // Android 10+：使用应用特定目录
        File(context.getExternalFilesDir(Environment.DIRECTORY_DOCUMENTS), filename)
    } else {
        // Android 9-：使用传统公共目录（需要权限）
        File(Environment.getExternalStoragePublicDirectory(
        Environment.DIRECTORY_DOCUMENTS),
        filename)
    }
}
```
##### 4.2 MediaStore API的版本适配
```kotlin
fun saveMediaFileCompat(context: Context, bitmap: Bitmap): Uri? {
    val contentValues = ContentValues().apply {
        put(MediaStore.Images.Media.DISPLAY_NAME, "image_${System.currentTimeMillis()}.jpg")
        put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")
    }

    return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        // Android 10+：使用RELATIVE_PATH
        contentValues.put(MediaStore.Images.Media.RELATIVE_PATH, 
            Environment.DIRECTORY_PICTURES + "/MyApp")
        contentValues.put(MediaStore.Images.Media.IS_PENDING, 1)
        
        val uri = context.contentResolver.insert(
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI, 
            contentValues
        )
        
        uri?.let {
            saveImageToUri(context, bitmap, it)
            // 标记完成
            contentValues.clear()
            contentValues.put(MediaStore.Images.Media.IS_PENDING, 0)
            context.contentResolver.update(it, contentValues, null, null)
        }
        uri
    } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
        // Android 6.0-9.0：需要检查权限
        if (checkStoragePermission(context)) {
            // 传统方式保存
            val uri = context.contentResolver.insert(
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI, 
                contentValues
            )
            uri?.let { saveImageToUri(context, bitmap, it) }
            uri
        } else {
            null
        }
    } else {
        // Android 5.1及以下
        val uri = context.contentResolver.insert(
            MediaStore.Images.Media.EXTERNAL_CONTENT_URI, 
            contentValues
        )
        uri?.let { saveImageToUri(context, bitmap, it) }
        uri
    }
}
```
##### 4.3 SAF的版本兼容性
Storage Access Framework自Android 4.4 (API 19)引入，向后兼容性较好，但需要注意持久化权限的处理：

```kotlin
fun handleUriPermission(context: Context, uri: Uri) {
// 检查是否有持久化权限
    val hasPersistedPermission = context.contentResolver.persistedUriPermissions.any { 
        it.uri == uri && it.isWritePermission 
    }

    if (!hasPersistedPermission && Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        try {
            context.contentResolver.takePersistableUriPermission(
                uri,
                Intent.FLAG_GRANT_READ_URI_PERMISSION or
                Intent.FLAG_GRANT_WRITE_URI_PERMISSION
            )
        } catch (e: SecurityException) {
            // 处理权限异常
            Log.e("Storage", "无法获取持久化权限", e)
        }
    }
}
```

#### 五、Android 13细化权限详解
##### 5.1 权限分组和变化
Android 13将原来的READ_EXTERNAL_STORAGE权限细化为三个独立权限：

```xml
<!-- AndroidManifest.xml声明 -->
<uses-permission android:name="android.permission.READ_MEDIA_IMAGES" />
<uses-permission android:name="android.permission.READ_MEDIA_VIDEO" />
<uses-permission android:name="android.permission.READ_MEDIA_AUDIO" />

<!-- 向后兼容：Android 12及以下使用旧权限 -->
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"
android:maxSdkVersion="32" />

```
##### 5.2 权限请求策略
```kotlin
class MediaPermissionManager(private val activity: Activity) {

    companion object {
        const val REQUEST_CODE_IMAGES = 1001
        const val REQUEST_CODE_VIDEO = 1002
        const val REQUEST_CODE_AUDIO = 1003
        const val REQUEST_CODE_ALL_MEDIA = 1004
    }
    
    /**
     * 请求特定媒体类型权限
     */
    fun requestSpecificMediaPermission(mediaType: MediaType) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            val permission = when (mediaType) {
                MediaType.IMAGE -> Manifest.permission.READ_MEDIA_IMAGES
                MediaType.VIDEO -> Manifest.permission.READ_MEDIA_VIDEO
                MediaType.AUDIO -> Manifest.permission.READ_MEDIA_AUDIO
            }
            
            if (ContextCompat.checkSelfPermission(activity, permission) 
                != PackageManager.PERMISSION_GRANTED) {
                
                ActivityCompat.requestPermissions(
                    activity,
                    arrayOf(permission),
                    when (mediaType) {
                        MediaType.IMAGE -> REQUEST_CODE_IMAGES
                        MediaType.VIDEO -> REQUEST_CODE_VIDEO
                        MediaType.AUDIO -> REQUEST_CODE_AUDIO
                    }
                )
            }
        } else {
            // Android 12及以下
            requestLegacyStoragePermission()
        }
    }
    
    /**
     * 请求所有媒体权限（适用于需要全部访问的应用）
     */
    fun requestAllMediaPermissions() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            val permissions = arrayOf(
                Manifest.permission.READ_MEDIA_IMAGES,
                Manifest.permission.READ_MEDIA_VIDEO,
                Manifest.permission.READ_MEDIA_AUDIO
            )
            
            ActivityCompat.requestPermissions(
                activity,
                permissions,
                REQUEST_CODE_ALL_MEDIA
            )
        } else {
            requestLegacyStoragePermission()
        }
    }
    
    /**
     * 检查是否有任何媒体权限
     */
    fun hasAnyMediaPermission(): Boolean {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            val permissions = arrayOf(
                Manifest.permission.READ_MEDIA_IMAGES,
                Manifest.permission.READ_MEDIA_VIDEO,
                Manifest.permission.READ_MEDIA_AUDIO
            )
            
            permissions.any { permission ->
                ContextCompat.checkSelfPermission(activity, permission) == 
                PackageManager.PERMISSION_GRANTED
            }
        } else {
            ContextCompat.checkSelfPermission(
                activity,
                Manifest.permission.READ_EXTERNAL_STORAGE
            ) == PackageManager.PERMISSION_GRANTED
        }
    }
    
    /**
     * 请求旧版本存储权限
     */
    private fun requestLegacyStoragePermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            ActivityCompat.requestPermissions(
                activity,
                arrayOf(Manifest.permission.READ_EXTERNAL_STORAGE),
                REQUEST_CODE_ALL_MEDIA
            )
        }
    }
    
    enum class MediaType {
        IMAGE, VIDEO, AUDIO
    }
}
```
##### 5.3 权限回调处理
```kotlin
class MainActivity : AppCompatActivity() {

    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        
        when (requestCode) {
            MediaPermissionManager.REQUEST_CODE_IMAGES -> {
                if (grantResults.isNotEmpty() && grantResults[0] == 
                    PackageManager.PERMISSION_GRANTED) {
                    // 图片权限已授予
                    loadImagesFromGallery()
                } else {
                    showPermissionDeniedDialog(getString(R.string.images_permission_denied))
                }
            }
            
            MediaPermissionManager.REQUEST_CODE_VIDEO -> {
                // 处理视频权限结果
            }
            
            MediaPermissionManager.REQUEST_CODE_AUDIO -> {
                // 处理音频权限结果
            }
            
            MediaPermissionManager.REQUEST_CODE_ALL_MEDIA -> {
                // 处理所有媒体权限结果
            }
        }
    }
    
    private fun showPermissionDeniedDialog(message: String) {
        AlertDialog.Builder(this)
            .setTitle(R.string.permission_required)
            .setMessage(message)
            .setPositiveButton(R.string.settings) { _, _ ->
                openAppSettings()
            }
            .setNegativeButton(R.string.cancel, null)
            .show()
    }
    
    private fun openAppSettings() {
        val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
            data = Uri.parse("package:$packageName")
        }
        startActivity(intent)
    }
}
```
#### 六、最佳实践和迁移指南
##### 6.1 版本适配策略
```kotlin
object StorageCompat {
    /**
     * 统一的文件保存方法，自动处理版本差异
     */
    fun saveFileCompat(
        context: Context,
        data: ByteArray,
        mimeType: String,
        displayName: String
    ): Uri? {
        return when {
            // Android 10+ 使用MediaStore
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q -> {
                saveViaMediaStore(context, data, mimeType, displayName)
            }
            
            // Android 6.0+ 检查权限后保存
            Build.VERSION.SDK_INT >= Build.VERSION_CODES.M -> {
                if (hasStoragePermission(context)) {
                    saveViaLegacyMethod(context, data, displayName)
                } else {
                    // 请求权限
                    requestStoragePermissionAndSave(context, data, displayName)
                }
            }
            
            // 旧版本直接保存
            else -> {
                saveViaLegacyMethod(context, data, displayName)
            }
        }
    }
    
    /**
     * 检查当前版本的存储权限状态
     */
    fun getStoragePermissionState(context: Context): PermissionState {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
            // Android 13+
            val hasImages = ContextCompat.checkSelfPermission(
                context, Manifest.permission.READ_MEDIA_IMAGES
            ) == PackageManager.PERMISSION_GRANTED
            
            val hasVideo = ContextCompat.checkSelfPermission(
                context, Manifest.permission.READ_MEDIA_VIDEO
            ) == PackageManager.PERMISSION_GRANTED
            
            val hasAudio = ContextCompat.checkSelfPermission(
                context, Manifest.permission.READ_MEDIA_AUDIO
            ) == PackageManager.PERMISSION_GRANTED
            
            PermissionState(hasImages, hasVideo, hasAudio)
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            // Android 6.0-12
            val hasStorage = ContextCompat.checkSelfPermission(
                context, Manifest.permission.READ_EXTERNAL_STORAGE
            ) == PackageManager.PERMISSION_GRANTED
            
            PermissionState(hasStorage, hasStorage, hasStorage)
        } else {
            // Android 5.1及以下，安装时已授予权限
            PermissionState(true, true, true)
        }
    }
    
    data class PermissionState(
        val canReadImages: Boolean,
        val canReadVideo: Boolean,
        val canReadAudio: Boolean
    )
}
```
