# Medium Clone

A Medium-style blogging application built with **Laravel 5.4**. Users can register, sign in, write rich-text articles with cover images, browse posts from other authors, and manage their own posts from a personal dashboard.

The project started as a Laravel "from scratch" blog and has been extended with image uploads, thumbnails, pagination, and post ownership checks.

## Features

- **Authentication** — registration, login, logout, and password reset (Laravel's built-in auth scaffolding).
- **Blog** — paginated list of posts, newest first.
- **Rich-text editor** — article bodies are written with CKEditor.
- **Cover images** — upload a cover image when creating or editing a post; an 80×80 thumbnail is generated automatically.
- **Post management** — authenticated users can create, edit, and delete their own posts.
- **Ownership protection** — editing and deleting are restricted to the post's author.
- **Dashboard** — a private page listing the signed-in user's posts with quick edit/delete actions.
- **Static pages** — Home, About, and Services.
- **Flash messages** — success/error feedback after actions.

## Tech Stack

| Layer | Technology |
| --- | --- |
| Backend | PHP, Laravel 5.4.24 |
| Database | MySQL / MariaDB |
| Frontend | Blade, Bootstrap 3, Vue 2, jQuery |
| Build tool | Laravel Mix (Webpack) 0.x |
| Editor | Unisharp Laravel CKEditor |
| Forms | Laravel Collective HTML |
| Testing | PHPUnit 5.7 |

## Project Structure

```
app/
  Http/Controllers/       PagesController, PostsController, DashboardController, Auth controllers
  Post.php                Post model (belongs to User)
  User.php                User model (has many Posts)
database/migrations/      users, password_resets, posts, user_id and cover_image migrations
resources/views/          Blade templates (layouts, pages, posts, auth, dashboard)
resources/assets/         JS and Sass sources compiled by Laravel Mix
routes/web.php            Application routes
_SQL/lsapp.sql            phpMyAdmin schema dump (optional import)
public/                   Web root and compiled assets
storage/                  Uploaded cover images and thumbnails
```

## Requirements

- PHP **7.1+** with the `openssl`, `pdo`, `mbstring`, `tokenizer`, `xml`, and `gd` extensions
- [Composer](https://getcomposer.org/)
- MySQL or MariaDB
- Node.js and npm (the frontend toolchain is from 2017, so a recent Node version may need compatibility flags)

> Laravel 5.4 is an old framework release. For best results use a matching legacy PHP version (7.1/7.2) rather than PHP 8.

## Getting Started

### 1. Install PHP dependencies

```bash
composer install
```

### 2. Configure the environment

```bash
cp .env.example .env
php artisan key:generate
```

Open `.env` and set your database credentials:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=medium_clone
DB_USERNAME=root
DB_PASSWORD=
```

### 3. Create the database schema

Either run the migrations:

```bash
php artisan migrate
```

Or import the included schema dump:

```bash
mysql -u root -p medium_clone < _SQL/lsapp.sql
```

### 4. Link public storage

Cover images are written to `storage/app/public/cover_images` and served from `/storage/cover_images`, so the storage symlink is required:

```bash
php artisan storage:link
```

### 5. Build the frontend assets

```bash
npm install
npm run dev
```

Use `npm run watch` while developing or `npm run production` for a minified build.

### 6. Serve the application

```bash
php artisan serve
```

Then open <http://localhost:8000>.

## Routes

| Method | URI | Action | Auth required |
| --- | --- | --- | --- |
| GET | `/` | Home page | No |
| GET | `/about` | About page | No |
| GET | `/services` | Services page | No |
| GET | `/posts` | List all posts | No |
| GET | `/posts/create` | New post form | Yes |
| POST | `/posts` | Store a post | Yes |
| GET | `/posts/{id}` | Show a post | No |
| GET | `/posts/{id}/edit` | Edit form (author only) | Yes |
| PUT/PATCH | `/posts/{id}` | Update a post (author only) | Yes |
| DELETE | `/posts/{id}` | Delete a post (author only) | Yes |
| GET | `/dashboard` | Current user's posts | Yes |
| — | `/login`, `/register`, `/password/*` | Authentication routes | No |

## Data Model

**User**
- `name`, `email`, `password`
- Has many `Post` records.

**Post**
- `title`, `body` (rich HTML), `cover_image`, `user_id`
- Belongs to a `User`.

Migrations create the schema incrementally:

1. `create_users_table`
2. `create_password_resets_table`
3. `create_posts_table`
4. `add_user_id_to_posts`
5. `add_cover_image_to_posts`

## Image Uploads

`PostsController` uses [Intervention Image](http://image.intervention.io/) to generate thumbnails, but the package is **not currently declared in `composer.json`**. If cover-image uploads fail, install and register it:

```bash
composer require intervention/image
```

Then add it to `config/app.php`:

```php
// providers
Intervention\Image\ImageServiceProvider::class,

// aliases
'Image' => Intervention\Image\Facades\Image::class,
```

CKEditor files are loaded from `public/vendor/unisharp/laravel-ckeditor/`. If that directory is missing, publish the package assets:

```bash
php artisan vendor:publish --provider="Unisharp\Ckeditor\ServiceProvider"
```

Uploaded files are stored in `storage/app/public/cover_images`; thumbnails are written to `storage/cover_images`. Make sure both locations are writable by the web server.

## Testing

The project includes Laravel's example tests:

```bash
vendor/bin/phpunit
```

## Notes

- New accounts can be created at `/register`; there are no database seeders configured.
- Uploaded images and post bodies are not sanitized beyond Laravel's validation, so only trusted users should be allowed to publish.
- The default `APP_NAME` is `Laravel`; change it in `.env` to rebrand the navbar and page title.
