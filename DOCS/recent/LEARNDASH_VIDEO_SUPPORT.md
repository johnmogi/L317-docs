# LearnDash Video Support - Proper LD30 Template Integration

This documentation explains the **correct approach** to add video support to custom LearnDash templates by using the LD30 template system properly, rather than bypassing it with workarounds.

## Problem

When using completely custom LearnDash lesson templates (outside the LD30 system), video embeds may not display properly because the templates bypass LearnDash's built-in video processing system that happens in `class-ld-cpt-instance.php`.

## The Correct Solution: LD30 Template Overrides

Instead of using mu-plugin workarounds, the proper approach is to create **custom LD30 template overrides** in your theme that work with LearnDash's native video processing system.

## How LearnDash Video Processing Works

1. **Video Processing**: LearnDash processes video content in `class-ld-cpt-instance.php` line 678:
   ```php
   $content = $ld_course_videos->add_video_to_content($content, $post, $lesson_settings);
   ```

2. **Template Loading**: The processed content (with video) is then passed to the LD30 template

3. **Template Display**: The LD30 template displays the content using the tabs system

## Files Created

### 1. `/wp-content/themes/your-theme/learndash/ld30/lesson.php`

Custom LD30 lesson template that:
- Receives the `$content` variable with video already processed by LearnDash
- Uses `learndash_get_template_part('modules/tabs.php')` to display content properly
- Maintains all LearnDash functionality (progression, assignments, navigation)
- Allows full customization of layout and styling

### 2. `/wp-content/themes/your-theme/learndash/ld30/topic.php`

Custom LD30 topic template with the same approach as lessons.

### 3. `/wp-content/mu-plugins/learndash-video-integration.php`

Supporting plugin that:
- Ensures `LEARNDASH_LESSON_VIDEO` constant is defined
- Enqueues video CSS and JavaScript assets when needed
- Provides debugging information
- Ensures video processing works with custom templates

## Key Benefits of This Approach

1. **Native Integration**: Works with LearnDash's built-in video processing system
2. **Full Functionality**: Maintains video progression tracking, resume functionality, and all video settings
3. **Asset Management**: Automatically loads required CSS and JavaScript
4. **Future-Proof**: Updates with LearnDash core updates
5. **Clean Code**: No workarounds or content filtering needed

## Template Structure

The custom LD30 templates follow this structure:

```php
<?php
// Custom lesson template: /themes/your-theme/learndash/ld30/lesson.php

// $content variable is automatically provided by LearnDash with video already processed
// $course_id, $user_id, $show_content, etc. are also provided

if ($show_content) {
    // Use LearnDash's tabs system which handles video content properly
    learndash_get_template_part(
        'modules/tabs.php',
        array(
            'content' => $content, // Already includes video!
            'materials' => $materials,
            'context' => 'lesson',
            // ... other parameters
        ),
        true
    );
}
?>
```

## Usage Instructions

### Step 1: Create Template Directory

Create the LearnDash template directory in your theme:
```
/wp-content/themes/your-theme/learndash/ld30/
```

### Step 2: Copy Template Files

The templates have been created at:
- `/themes/hello-elementor/learndash/ld30/lesson.php`
- `/themes/hello-elementor/learndash/ld30/topic.php`

### Step 3: Customize as Needed

Edit the templates to match your design requirements. The key parts to maintain:

```php
// This line displays content with video already processed
learndash_get_template_part(
    'modules/tabs.php',
    array(
        'content' => $content, // Video is already in here!
        'materials' => $materials,
        'context' => 'lesson',
    ),
    true
);
```

### Step 4: Test Video Functionality

1. Go to a lesson in WordPress admin
2. Add a video URL in the "Video" section
3. View the lesson on the frontend
4. The video should display properly with progression tracking

## Template Customization Examples

### Adding Custom Header

```php
<div class="custom-lesson-header">
    <h1 class="lesson-title"><?php the_title(); ?></h1>
    <!-- Add breadcrumbs, progress bar, etc. -->
    
    <?php if (learndash_has_video()): ?>
        <div class="lesson-video-section">
            <h2>Lesson Video</h2>
            <?php learndash_display_video(); ?>
        </div>
    <?php endif; ?>
    
    <div class="lesson-text-content">
        <?php the_content(); ?>
    </div>
</div>

<?php get_footer(); ?>
```

### Advanced Template with Video Controls

```php
<?php
// Custom video display with wrapper
$video_args = array(
    'wrapper_class' => 'custom-video-container',
    'before_video' => '<div class="video-title">Watch the Lesson</div>',
    'after_video' => '<div class="video-controls">Video controls here</div>',
);

if (learndash_has_video()) {
    echo learndash_get_video_html($video_args);
}
?>
```

## Available Functions

### Core Functions

- `learndash_has_video($post_id = null)` - Check if lesson/topic has video
- `learndash_get_video_content($post_id = null)` - Get raw video content
- `learndash_display_video($args = array())` - Display video HTML
- `learndash_get_video_html($args = array())` - Get video HTML string

### Information Functions

- `learndash_get_video_url()` - Get video URL
- `learndash_get_video_provider()` - Get provider (youtube, vimeo, etc.)
- `learndash_get_video_settings()` - Get all video settings
- `learndash_is_video_progression_enabled()` - Check if progression tracking enabled

### Debug Function

- `learndash_video_debug_info()` - Display debug information (admin only)

## Video Features Supported

✅ **YouTube Videos** - Full support with progression tracking  
✅ **Vimeo Videos** - Full support with progression tracking  
✅ **Wistia Videos** - Full support with progression tracking  
✅ **Local Videos** - MP4, WebM, etc.  
✅ **Video Progression** - Track viewing time and completion  
✅ **Auto-play Settings** - Respect LearnDash video settings  
✅ **Video Controls** - Show/hide controls based on settings  
✅ **Responsive Design** - Videos adapt to container size  

## Video Settings (LearnDash Admin)

All standard LearnDash video settings are supported:

- **Video URL** - YouTube, Vimeo, Wistia, or direct video URLs
- **Auto Start** - Automatically start video when page loads
- **Show Controls** - Display video player controls
- **Track Time** - Enable video progression tracking
- **Auto Complete** - Mark lesson complete when video ends
- **Auto Complete Delay** - Delay before auto-completion
- **Hide Complete Button** - Hide until video is watched
- **Focus Pause** - Pause when user leaves tab/window

## Troubleshooting

### Videos Not Showing

1. **Check Video Settings**: Ensure video is enabled in lesson settings
2. **Check Video URL**: Verify the video URL is correct and accessible
3. **Check Debug Info**: Add `<?php echo learndash_video_debug_info(); ?>` to your template

### Video Progression Not Working

1. **Check Settings**: Ensure "Track Time" is enabled in lesson video settings
2. **Check JavaScript**: Ensure jQuery is loaded and no JavaScript errors
3. **Check User Login**: Video progression requires logged-in users

### CSS Styling Issues

The plugin automatically loads LearnDash video CSS. You can override styles:

```css
/* Custom video styling */
.ld-video {
    margin: 20px 0;
    border-radius: 8px;
    overflow: hidden;
}

.ld-video iframe {
    width: 100%;
    height: auto;
    aspect-ratio: 16/9;
}
```

## Debug Mode

For troubleshooting, the plugin includes debug information when `WP_DEBUG` is enabled. Check your browser's HTML source for debug comments.

## Example: Complete Custom Lesson Template

```php
<?php
/**
 * Custom LearnDash Lesson Template with Video Support
 */
get_header(); ?>

<div class="custom-lesson-container">
    <div class="lesson-header">
        <h1 class="lesson-title"><?php the_title(); ?></h1>
        
        <?php if (learndash_has_video()): ?>
            <div class="lesson-video-indicator">
                <span class="video-icon">📹</span>
                <span>This lesson includes a video</span>
            </div>
        <?php endif; ?>
    </div>
    
    <div class="lesson-content-wrapper">
        <?php if (learndash_has_video()): ?>
            <div class="lesson-video-section">
                <h2>Lesson Video</h2>
                <div class="video-info">
                    <p>Provider: <?php echo learndash_get_video_provider(); ?></p>
                    <?php if (learndash_is_video_progression_enabled()): ?>
                        <p><em>Video progress is tracked for this lesson</em></p>
                    <?php endif; ?>
                </div>
                
                <?php 
                learndash_display_video(array(
                    'wrapper_class' => 'custom-video-wrapper',
                    'before_video' => '<div class="video-intro">Watch this lesson:</div>',
                    'after_video' => '<div class="video-note">Complete the video to continue</div>'
                )); 
                ?>
            </div>
        <?php endif; ?>
        
        <div class="lesson-text-content">
            <?php the_content(); ?>
        </div>
    </div>
    
    <?php if (current_user_can('manage_options')): ?>
        <div class="admin-debug">
            <?php echo learndash_video_debug_info(); ?>
        </div>
    <?php endif; ?>
</div>

<?php get_footer(); ?>
```

## Support

The video support works exactly like the legacy LearnDash theme. Your YouTube video example:

```html
<div class="ld-video" data-video-progression="false" data-video-cookie-key="learndash-video-progress-505d6af87cc1819af64ed85e41eb0619" data-video-provider="youtube">
    <iframe title="שיעור 12 מערכת האורות ותחזוקת הרכב" width="800" height="450" src="https://www.youtube.com/embed/t-9JN9PHZzM?feature=oembed" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen=""></iframe>
</div>
```

Will now be generated automatically in your custom templates with all the same functionality, progression tracking, and styling.
