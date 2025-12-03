# XML Feed Widget Extension

A configurable dashboard widget that displays XML/RSS feed items in the PyroCMS control panel.

## Description

This extension provides a dashboard widget that fetches and displays items from any XML or RSS feed. By default, it displays recent blog posts from PyroCMS.com, but can be configured to show any compatible feed source. Perfect for keeping up with news, blog posts, announcements, or any RSS/Atom feed directly from your dashboard.

## Features

- **RSS/Atom Support**: Compatible with standard XML feed formats
- **Configurable URL**: Set any feed URL per widget instance
- **Caching**: Built-in 30-minute cache to reduce external requests
- **Fallback Mechanisms**: Multiple fetch methods for maximum compatibility
- **SSL Friendly**: Works with HTTPS feeds and SSL certificates
- **Limit Control**: Displays the 5 most recent items
- **SimplePie Integration**: Uses the robust SimplePie library for feed parsing

## Installation

This extension is typically included with PyroCMS. If you need to install it separately:

```bash
composer require anomaly/xml_feed_widget-extension
```

## Usage

### Adding the Widget to Your Dashboard

1. Navigate to the Dashboard in the PyroCMS control panel
2. Click "Customize Dashboard" or "Add Widget"
3. Select "XML Feed" from the available widgets
4. Configure the feed URL
5. Save and position the widget on your dashboard

### Configuration

Each widget instance can be configured with:

- **URL**: The XML/RSS feed URL (default: `http://pyrocms.com/posts/rss.xml`)

#### Example Feed URLs

```
# PyroCMS Blog
http://pyrocms.com/posts/rss.xml

# Laravel News
https://feed.laravel-news.com/

# Any WordPress Blog
https://yourblog.com/feed/

# Medium Publication
https://medium.com/feed/@username
```

## How It Works

### Fetch Process

The widget uses a smart fallback system to fetch feeds:

1. **Raw Content Method** (Primary)
   - Uses `file_get_contents()` with SSL verification disabled
   - More SSL/TLS friendly with CDNs like CloudFlare
   - Falls back if blocked by security settings

2. **cURL Method** (Fallback)
   - Uses SimplePie's built-in cURL fetching
   - Works when `file_get_contents()` is disabled
   - Handles SSL/TLS certificates automatically

3. **Error Handling**
   - Returns `false` if both methods fail
   - Widget displays gracefully without breaking

### Caching

- **Duration**: 30 minutes per widget instance
- **Key**: Unique per widget ID
- **Storage**: Uses Laravel's configured cache driver
- **Benefits**: Reduces external API calls and improves performance

## Code Examples

### Programmatic Widget Creation

```php
use Anomaly\DashboardModule\Widget\Contract\WidgetRepositoryInterface;

$widgets = app(WidgetRepositoryInterface::class);

$widget = $widgets->create([
    'extension' => 'anomaly.extension.xml_feed_widget',
    'title'     => 'Laravel News',
    'dashboard' => $dashboard->getId(),
]);

// Configure the feed URL
app('Anomaly\ConfigurationModule\Configuration\Contract\ConfigurationRepositoryInterface')
    ->create([
        'scope' => $widget->getId(),
        'key'   => 'anomaly.extension.xml_feed_widget::url',
        'value' => 'https://feed.laravel-news.com/',
    ]);
```

### Accessing Feed Items in Views

```twig
{# In your widget view #}
{% if items %}
    <ul class="feed-items">
        {% for item in items %}
            <li>
                <a href="{{ item.get_permalink() }}" target="_blank">
                    {{ item.get_title() }}
                </a>
                <small>{{ item.get_date('j F Y') }}</small>
            </li>
        {% endfor %}
    </ul>
{% else %}
    <p>Unable to load feed items.</p>
{% endif %}
```

## SimplePie Methods

The widget returns SimplePie items, which provide many useful methods:

```php
// Title
$item->get_title()

// Description/Content
$item->get_description()
$item->get_content()

// Link
$item->get_permalink()
$item->get_link()

// Date
$item->get_date('j F Y')
$item->get_gmdate('Y-m-d')

// Author
$item->get_author()

// Categories
$item->get_categories()
```

Full SimplePie documentation: https://simplepie.org/api/

## Use Cases

### News Aggregation
Display industry news, blog posts, or announcements relevant to your team.

### Internal Updates
Show company blog updates, release notes, or internal announcements.

### Monitoring
Keep track of status pages, changelog feeds, or service updates.

### Content Curation
Display curated content from multiple sources on different dashboard widgets.

### Team Communication
Share RSS feeds from project management tools, forums, or communication platforms.

## Troubleshooting

### Feed Not Loading

**Check URL**: Ensure the feed URL is valid and accessible
```bash
curl -I https://yourfeed.com/rss.xml
```

**SSL Issues**: If using HTTPS, verify the certificate is valid

**Server Configuration**: Ensure `allow_url_fopen` is enabled or cURL is available
```php
// Check PHP settings
var_dump(ini_get('allow_url_fopen'));
var_dump(function_exists('curl_init'));
```

**Clear Cache**: Force refresh by clearing the cache
```bash
php artisan cache:clear
```

### Performance Issues

- **Reduce Cache Time**: Modify cache duration in `LoadItems.php`
- **Check Feed Size**: Large feeds may take longer to parse
- **Network Latency**: Slow external servers affect load times

## Security Considerations

### SSL Verification
By default, SSL verification is disabled for compatibility. To enable:

```php
// Modify FetchRawContent.php
$options = [
    'ssl' => [
        'verify_peer'      => true,
        'verify_peer_name' => true,
    ],
];
```

### Trusted Sources
Only configure feeds from trusted sources to prevent:
- Malicious content injection
- Tracking or privacy concerns
- Performance degradation

## Requirements

- PyroCMS 3.x
- Anomaly Streams Platform ^1.8
- SimplePie ^1.5
- PHP `allow_url_fopen` enabled OR cURL extension

## Dependencies

- **SimplePie**: RSS/Atom feed parser
- **Dashboard Module**: For widget integration
- **Configuration Module**: For URL settings

## Support

- **Email**: support@anomaly.is
- **Website**: http://pyrocms.com/
- **Documentation**: [PyroCMS Documentation](https://pyrocms.com/documentation)

## License

This extension is open-sourced software licensed under the [MIT license](LICENSE.md).

## Authors

- **PyroCMS, Inc.** - [Website](http://pyrocms.com/) - support@pyrocms.com
