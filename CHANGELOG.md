# Changelog

All notable changes to the unnsse.io blog will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-09-05

### Added
- Created `/layouts/robots.txt` with comprehensive SEO-friendly configuration
  - Allows major search engines (Google, Bing) to crawl content
  - Blocks unwanted bots and crawlers (MJ12bot, AhrefsBot, BLEXBot, etc.)
  - Protects static assets (`/images/`, `/js/`, `/css/`)
  - Includes proper sitemap reference
- Added this CHANGELOG.md file to track project changes

### Fixed
- **CRITICAL SEO FIX**: Resolved sitemap URL issue
  - Fixed sitemap generation to use production URLs (`https://unnsse.io/`) instead of localhost URLs (`http://localhost:1313/`)
  - Ran production Hugo build to regenerate correct sitemap with proper domain
  - All article URLs now properly indexed for search engines

### Technical Details
- Used FixIt theme's robots.txt as base template, copying all relevant SEO configurations
- Verified `enableRobotsTXT = true` is set in configuration files
- Project-level `/layouts/robots.txt` now takes precedence over theme's robots.txt
- Sitemap configuration remains optimal:
  - Weekly change frequency
  - Priority: 0.5
  - Proper XML structure with lastmod dates

### Impact
- Google and other search engines should now be able to properly crawl and index individual articles
- Previously, articles were not appearing in Google search results due to localhost URLs in sitemap
- DuckDuckGo/Bing indexing was working correctly and should continue to work