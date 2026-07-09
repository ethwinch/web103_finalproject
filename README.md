# Collector's Corner

CodePath WEB103 Final Project

Designed and developed by: Sam Kosal, Teo Liang, Jianxin Lin, Hien Vo Minh, Ethan Winch

🔗 Link to deployed app: TBD (Render link will go here)

## About

### Description and Purpose

<p>This app provides users with the ability to create a media library for their media collection and browse the collections of others. Media can include books, movies, music, magazines, audio books, and videos.</p>

<p>This is meant to act as a “book shelf” to display whatever media a user decides to display. It can be used to create watch lists to share among friends, to keep track of all the media one has consumed, to create recommendation lists, gather a collection of media that fits a certain theme, or whatever other purpose users can think to give it.</p>

<p>It’s a great way to keep track of your favorite media across platforms, to share the media you love with others, discover new media, and more.</p>


### Inspiration

<p>I have a lot of physical media that is not available online anywhere or is one-of-a-kind, and having a space to digitally catalogue that to both share with others and also track what I own is helpful and also a fun way to connect with others.</p>

<p>On many indie websites and blogs, one of the things people tend to share the most is the music they listen to, books they read, or shows they watch. I’ve seen many unique ways of displaying their favorite albums, books and movies, but wouldn’t it be nice to have a place for it all that’s easy to discover and share with others? Plenty of people share their reading lists on Goodreads or StoryGraph, but what if you want to share more than just books?</p>

<p>I’ve been using my own blog to track the books I read in a fun skeuomorphic way that emulates a bookshelf, and I wanted to take it a step further and expand on that one feature until it can become a whole app.</p>


## Tech Stack

Frontend:

Backend:

## Features

### Add Custom Media

<p>Ability to upload an image, create tags, and add a title and description of the media you want to add to your collection. (Allows for niche or personal media to be added and not only recognized/published media.)</p>

[gif goes here]

### Search Pre-existing Media

<p>The app remembers the custom media others have uploaded, so if a user wants to add media that already exists on the site, they are able to search for and add it to their collection without having to manually create a new media, upload an image, fill out a description, etc. </p>

[gif goes here]

### Sort Media

<p>Filter the media within a collection by tags.</p>

[gif goes here]

### Link to Media

<p>Allow users to attach a link to the media. This link can send users to the streaming service it’s hosted on, a link to where it’s hosted and available to watch, or a related site.</p>

[gif goes here]

### Browse Collections

<p>Browse other’s media collections they have made.</p>

[gif goes here]

### Filter Collections

<p>Filter media collections by date, type (books, music, films), and tags.</p>

[gif goes here]

### Custom Tags

<p>Users can create custom tags to sort their collection by instead of being stuck with predefined ones.</p>

[gif goes here]

### Access Control

<p>Users can only create/update/delete media from a collection if they are logged into the account associated with that collection.</p>

[gif goes here]

<!--
### [ADDITIONAL FEATURES GO HERE - ADD ALL FEATURES HERE IN THE FORMAT ABOVE; you will check these off and add gifs as you complete them]
-->

## Installation Instructions

[instructions go here]

---

## Milestone 1 Technical Additions

### Tech Stack (Detailed)

**Frontend**
- React (component-based UI)
- React Router (dynamic routing, nested/protected routes)
- Fetch or Axios (API communication)
- CSS Modules or plain CSS (styling)
- Optional UI utility library for modal/slide-out panel interactions

**Backend**
- Node.js + Express.js (REST API and server-side logic)
- express-session + connect-pg-simple (session-based authentication stored in PostgreSQL)
- Multer (multipart/form-data handling for image uploads)
- Cloudinary SDK (media cover image hosting)

**Database**
- PostgreSQL
- SQL migrations (recommended: node-pg-migrate/Knex/Drizzle migration flow)

**Deployment**
- Render (frontend and backend services)
- Managed PostgreSQL (Render Postgres or equivalent)
- Cloudinary (external asset storage)

### Feature Requirements Breakdown

#### Baseline Features (Required)
1. **Express.js backend + React frontend with dynamic routes**
   - Backend routes will be namespaced under `/api/*`.
   - Frontend routes include public browse routes and protected personal management routes.
2. **RESTful API with GET, POST, PATCH, DELETE**
   - CRUD flows for media, tags, library entries, and profiles.
3. **One-to-many + many-to-many relationships**
   - One-to-many: users→library_entries, media→library_entries, users→tags.
   - Many-to-many: library_entries↔tags through `library_entry_tags`.
4. **Dynamic routing with React Router**
   - Example patterns: `/media/:mediaId`, `/users/:userId/collections`, `/library/:entryId/edit`.
5. **Hierarchical React component design**
   - Route-level pages compose reusable components (cards, filters, modals, forms).
6. **At least one redirection + one same-page interaction**
   - Redirection: after login/logout and after create/edit flows.
   - Same-page interaction: tag filter/sort updates results without full page reload.
7. **Deploy on Render**
   - Final production deployment will include environment-based API/asset configuration.

#### Custom Features (All 5 Planned)
1. **Validation before POST/PATCH persistence**
   - Server validates required fields, enum constraints, ownership, and relationship existence.
2. **Filtering/sorting by custom tags**
   - Users can filter their collection by one or multiple custom tags and media type.
3. **Slide-out pane or modal for create/edit**
   - Reusable modal system for media entries, library entries, and tag management.
4. **Graceful error handling**
   - Consistent backend error payload shape and user-friendly frontend error states.
5. **One-to-one users↔user_profiles**
   - Every registered user has exactly one profile row with display metadata.

#### Stretch Features (All 5 Planned)
1. **Authentication + Protected Routes**
   - Session cookie auth gates personal pages and mutating actions.
2. **Role and ownership restrictions**
   - Visitor/Collector/Creator/Curator capabilities enforced in API and UI.
3. **Loading spinners**
   - Async pages show loading states during fetch and mutation requests.
4. **Toast feedback**
   - Success/error toasts for create, update, delete, login/logout actions.
5. **Cloud image upload**
   - Creators (and Curators if allowed) can upload covers to Cloudinary.

### Role Summary

- **Visitor (Guest)**: Browse public collections and global media catalog in read-only mode.
- **Collector (Authenticated)**: Build and organize personal library, manage personal tags.
- **Creator (Authenticated)**: Collector permissions plus ability to create global media entries and upload covers.
- **Curator (Authenticated)**: Emphasis on curation quality, organization, and surfacing valuable content.

### API and Architecture Direction (Milestone 1 Plan)

- API will enforce both **authentication** (is logged in) and **authorization** (owns resource or has role privilege).
- Mutating endpoints will check ownership for `library_entries` and `tags` to prevent cross-user modification.
- Tagging design uses a join table to support multi-tag assignment per library item while preventing duplicates.
- Frontend state will separate global catalog browsing from personal collection management for clarity and security.
