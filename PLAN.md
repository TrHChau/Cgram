# Cgram — Product specification and implementation roadmap

Status: planning only; application implementation has not started.
Last updated: 2026-11-07.

This document preserves the product interview and organizes future implementation into stages. Creating this document does not authorize implementing the app, provisioning services, or publishing builds. Start implementation only when requested.

## 1. Product goal

Build a simple Instagram-style mobile app for a small invited community to share photos and videos, follow people they know, and exchange direct messages.

- Initial audience: up to 25 friends, classmates, and coworkers in Vietnam, including ages 13–17 and adults.
- Initial platform: iOS, distributed through TestFlight.
- Interface: English; user-generated content can use other languages.
- Design: modern, clean, minimal, mobile-first, neutral black/white/gray palette, following system light/dark mode.
- Delivery priority: complete the agreed v1 rather than target a fixed launch date.
- Future direction: Android next; group chats and stories later.

## 2. Confirmed technology and project constraints

- Expo / React Native with TypeScript.
- Expo Router for all navigation; four native tabs.
- NativeWind for styling.
- Clerk for authentication through Google and Apple.
- Convex for backend functions and application database; investigate Convex file storage first.
- Expo/EAS for cloud builds and TestFlight delivery.
- Set up Apple Developer, Expo/EAS, Clerk, and Convex; none should be assumed ready.
- Keep service usage on free tiers initially. Apple Developer membership is an accepted separate expense.
- Do not automatically upgrade to paid hosting. Pause uploads at configured capacity limits.

### Repository baseline

At planning time, `package.json` declares Expo `~58.0.23`, Expo Router `~58.0.30`, React `19.3.0`, and React Native `0.88.1`. These are manifest declarations, not a guarantee of future resolved versions or compatibility.

The repository uses `package-lock.json`; no Bun lockfile was observed. The app currently has a root stack layout and a placeholder index screen. Clerk, Convex, and NativeWind are not yet declared dependencies. Existing user changes must be preserved.

### Rules for future implementation

- Read `AGENTS.md` before implementation.
- Read the installed Expo major version and matching versioned documentation before touching Expo or React Native APIs.
- Consult Expo's documentation index for the specific API being used; do not rely on remembered APIs.
- Use `npx expo install <package>` for SDK-compatible dependency installation. If the project later adopts Bun, use `bunx` as instructed in `AGENTS.md`.
- Keep route files in `src/app/`; keep components, hooks, utilities, and business logic outside that directory.
- Do not manually create or edit generated `ios/` or `android/` directories; use app configuration and config plugins.
- Use development builds when required by native integrations.
- Run lint and TypeScript checks during implementation and before declaring implementation work complete.

## 3. Confirmed v1 experience

### Authentication and onboarding

1. A signed-out user sees Google and Apple sign-in options.
2. Clerk authenticates the user.
3. First-time users choose a unique username and account visibility, with private preselected.
4. Suggest an editable display name; avatar and bio are optional.
5. Completed onboarding leads to Home. Returning users bypass onboarding when their profile is complete.

Account privacy must be enforced by the backend, not only by hidden UI.

### Main tabs

| Tab | Primary behavior |
| --- | --- |
| Home | Chronological feed of the user's own posts and posts from accounts they follow and may access. Newest first. Header plus button opens post creation. |
| Messages | One-to-one conversations, real-time messages, and unread indicators. |
| Explore | Recent public posts and people search by username or display name. Private posts do not appear here. |
| Profile | Own profile, posts, profile editing, privacy/settings, and access to relevant account controls. |

New users with an empty Home feed see a clear prompt linking to people discovery in Explore. Do not fill Home with an unrequested algorithmic recommendation feed.

### Post creation and display

- Select existing photos and videos from the device library.
- A post contains one to five media items, including mixed image/video carousels.
- Videos are limited to 30 seconds each.
- Preview and reorder selected media before publishing.
- Allow an optional caption.
- Support upload progress, validation errors, failures, and retry.
- Users can like/unlike posts and delete their own posts.
- No in-app camera, image cropping, filters, or video trimming in v1.
- No post editing after publication in v1.
- Exact byte-size limits and supported media encodings remain to be validated during the media feasibility stage.

### Comments

- Flat chronological comments; no threads, replies, or comment likes.
- Users can add comments to posts they are allowed to access.
- Authors can delete their own comments.
- Moderators can remove reported content through the moderation workflow.

### Profiles, following, and privacy

- Users can view and edit their profile.
- Public accounts accept follows immediately.
- Private accounts require follow approval.
- Non-followers viewing a private account see username, display name, avatar, bio, and counts.
- Private posts and follower/following lists require approved access.
- Owners can approve/reject follow requests and remove followers.
- Switching public to private preserves existing followers; only new followers require approval.
- Include in-app indicators for pending follow requests.

### Direct messages

- One-to-one text messages and photo/video attachments.
- One attachment per message; videos remain capped at 30 seconds.
- Real-time conversation updates and unread indicators.
- Both people must follow each other to send messages.
- Unfollowing or blocking stops further messages.
- Existing history and attachments remain readable to conversation participants after unfollowing or blocking.
- Do not restore sending while a block exists, even if follow records remain.
- No group chats, message requests, typing indicators, read receipts, message unsending, or inbox/conversation deletion in v1.

### Media authorization

- Private-post media and DM attachments require authorization on every delivery request.
- Removing a follower or otherwise revoking post access must prevent subsequent authorized delivery of private-post media.
- DM authorization follows conversation membership and the agreed history-retention behavior: a block stops sending, but does not revoke old attachment access from participants.
- Account/content deletion overrides ordinary history retention and removes owned media.
- Authorization cannot retract copies a recipient has already downloaded or captured.

### Blocking, reports, and moderation

- Users can block accounts and report posts, comments, and messages.
- Provide an admin-only screen to review reports, dismiss reports, remove content, and suspend accounts.
- Enforce moderator permissions on the backend.
- Report review should reveal only the content and context necessary for moderation, not unrestricted browsing of private conversations.
- Exact non-messaging effects of blocking must be finalized before implementation; see proposed defaults below.

### Account deletion

- Allow users to delete their account.
- Remove their profile, posts, comments, sent message text, and owned attachments.
- Recipients may see a generic deleted-content placeholder where needed to preserve understandable history.
- Coordinate Clerk identity deletion with Convex data and file cleanup.
- Handle interruption and retries so partial deletion cannot leave an active account with inconsistent access.

### Connectivity and notifications

- Online-first app with clear connection, loading, empty, error, and retry states.
- No guaranteed offline reading, offline publishing, persistent offline drafts, or queued offline sending.
- In-app unread and follow-request indicators only; no push notifications in v1.

## 4. Explicitly out of scope

- Android and web releases in v1.
- Group chats and stories.
- Monetization, subscriptions, and ads.
- Push notifications, typing indicators, and read receipts.
- Message requests, unsending, and hiding/deleting inbox conversations.
- Camera capture, filters, crop, and trim tools.
- Post editing, hashtags, user tagging, bookmarks, and sharing posts inside DMs.
- Algorithmic content ranking, deep comment threads, and comment likes.
- Advanced offline synchronization.

Do not add these features implicitly while implementing adjacent functionality.

## 5. Proposed architecture and data responsibilities

This section is an implementation direction, not a finalized schema or a claim that SDK integrations have been validated.

### Boundaries

- Mobile client: rendering, navigation, form state, device media selection, upload progress, and user feedback.
- Clerk: identity, provider sign-in, sessions, and authentication lifecycle.
- Convex: application records, authorization, validation, social operations, message persistence, reactive reads, and cleanup orchestration.
- Storage/delivery: Convex storage is the first option to evaluate; protected delivery must satisfy the agreed authorization rules.
- EAS: development/distribution builds and release delivery.

Never trust client-supplied user IDs, owner flags, admin flags, or visibility claims. Derive the actor from verified authentication and enforce access in backend functions.

### Logical records

| Record | Intended information and relationships |
| --- | --- |
| Profile | Clerk identity reference, unique normalized username, display name, avatar reference, bio, privacy, moderation status, onboarding status, timestamps. |
| Follow relationship | Follower and target; pending or accepted state. One logical relationship per ordered user pair. |
| Post | Author, caption, ordered media references, publication state, timestamps. Access follows the author's account visibility. |
| Media | Owner, storage reference, media type, dimensions/duration/size as applicable, lifecycle state, and owning post/message/profile. |
| Like | User and post with uniqueness per pair. |
| Comment | Author, post, text, timestamp. |
| Conversation | One-to-one participant pair and metadata needed for ordering and unread state. |
| Message | Conversation, sender, optional text/media, timestamp, and retry/deduplication identifier. |
| Conversation membership | Participant authorization and last-read position for unread indicators; not recipient-visible read receipts. |
| Block | Blocking user and blocked user, unique per ordered pair. |
| Report | Reporter, target type/reference, reason, status, review metadata, and only necessary evidence. |
| Moderation action | Moderator, action, target, timestamp, and audit information. |
| Cleanup job | Retryable account/content deletion progress and pending file cleanup. |

Define concrete indexes, validation lengths, pagination, rate limits, and deletion relationships before implementing each subsystem. Avoid collecting extra personal information such as birth dates unless the audience/privacy review establishes a need.

### Backend behavior to preserve

- Paginate feeds, comments, member lists, and messages; do not download entire tables to the client.
- Enforce unique usernames, follow relationships, likes, and one-to-one conversations under concurrent requests.
- Recheck access for direct post/profile/conversation lookups, not only list queries.
- Authenticate media delivery and prevent unauthorized reuse of protected media URLs.
- Publish a post only when all its media is successfully uploaded and validated.
- Clean up orphaned uploads and retry deletion work safely.
- Prevent duplicate posts/messages when clients retry after an ambiguous network failure.
- Revalidate active access when privacy, follow status, blocks, suspension, or account deletion changes.

## 6. Step-by-step implementation roadmap

All checkboxes are intentionally unchecked. Complete stages in dependency order, recording findings and deviations in this document.

### Stage 0 — Validate feasibility and settle remaining decisions

- [ ] Inspect the current repository, preserve user changes, and read `AGENTS.md`.
- [ ] Confirm exact resolved package versions and current SDK 58 documentation.
- [ ] Validate Clerk, NativeWind, native tabs, media picker, and video playback compatibility before selecting versions.
- [ ] Prototype the design for authenticated image/video delivery, including video seeking/range requests, request credentials, and caching behavior.
- [ ] Confirm that copied protected URLs cannot bypass authorization and that revoked post access affects subsequent requests.
- [ ] Evaluate Convex storage, delivery bandwidth, function execution, and free-tier restrictions for 25 media-sharing testers.
- [ ] Choose explicit image/video byte limits and supported formats; define rejection behavior for unsupported or oversized media.
- [ ] Establish app-wide upload limits, usage monitoring, and an upload kill switch. Account for download traffic independently.
- [ ] Review requirements applicable to teenage testers in Vietnam, including privacy, consent, reporting, and distribution. Do not assume an adults-only audience.
- [ ] Resolve proposed defaults in section 7 where they materially affect behavior.

Acceptance: document a feasible media/privacy/cost approach. If the chosen stack cannot meet the requirements, present the concrete tradeoff before weakening privacy or adding paid services.

### Stage 1 — Accounts, configuration, and app foundation

- [ ] Set up owner-controlled Apple Developer, Expo/EAS, Clerk, and Convex accounts/projects.
- [ ] Configure app identifiers, sign-in providers, redirect handling, and required capabilities using current documentation.
- [ ] Separate development and release configuration; keep secrets out of source control and client bundles.
- [ ] Install compatible dependencies using the project's Expo installation rules.
- [ ] Configure NativeWind and shared light/dark design tokens.
- [ ] Build the four native tabs with nested detail/compose/settings navigation.
- [ ] Establish reusable loading, empty, error, form, and accessible interaction patterns.
- [ ] Configure development builds and lint/typecheck tooling without silently changing unrelated user work.

Acceptance: a development build launches on a real iPhone; native tabs, theme switching, navigation, lint, and typecheck work.

### Stage 2 — Authentication, profiles, and onboarding

- [ ] Integrate Google and Apple authentication with Clerk and verified Convex identity.
- [ ] Implement idempotent profile creation and incomplete-onboarding recovery.
- [ ] Add unique username selection, display name, optional avatar/bio, and privacy selection.
- [ ] Implement sign-out, returning-user routing, and profile viewing/editing.
- [ ] Define account suspension checks and moderator role provisioning securely.

Acceptance: both providers work on device; duplicate usernames are rejected; interrupted onboarding resumes; users cannot edit someone else's profile.

### Stage 3 — Follow graph and visibility

- [ ] Implement public follows, private follow requests, approval/rejection, unfollow, and follower removal.
- [ ] Implement private profile redaction and authorized follower/following lists.
- [ ] Add in-app pending-request indicators and a request-management screen.
- [ ] Implement account visibility changes without removing existing approved followers.
- [ ] Centralize visibility checks for reuse by posts, media, and messaging.

Acceptance: exercise public/private profiles with owner, follower, pending requester, unrelated user, blocked user, and moderator identities.

### Stage 4 — Media pipeline and posting

- [ ] Implement library permission handling, selection, preview, ordering, and caption entry.
- [ ] Validate count, duration, file size, and format using the feasibility-stage decisions.
- [ ] Implement authenticated upload authorization, progress, retry, cancellation handling, and cleanup.
- [ ] Implement atomic publication after all selected media succeeds.
- [ ] Implement protected delivery, image rendering, carousel paging, and efficient video playback.
- [ ] Add owner-authorized post deletion and storage cleanup.

Acceptance: mixed five-item posts work; invalid uploads fail clearly; partial uploads do not publish partial posts; revoked access and deleted media cannot be fetched through protected delivery.

### Stage 5 — Home, Explore, likes, and comments

- [ ] Build the paginated chronological Home feed, including own posts and empty-feed discovery prompt.
- [ ] Build recent public-post Explore and username/display-name search.
- [ ] Add post detail, likes/unlikes, flat comments, and own-comment deletion.
- [ ] Handle access revocation, removed posts, and concurrent interactions without stale unauthorized content.
- [ ] Validate scrolling, image sizing, video resource use, and keyboard behavior on device.

Acceptance: private posts never leak through Explore, search, direct lookup, or stale access paths; repeated like operations cannot create duplicate likes.

### Stage 6 — One-to-one messaging

- [ ] Create or reuse one conversation per participant pair.
- [ ] Add conversation list, paginated history, live updates, and unread state.
- [ ] Implement text and single-attachment sending with retry deduplication.
- [ ] Enforce mutual follows and absence of blocks at send time.
- [ ] Preserve read access to existing history after unfollow/block while disabling sending.
- [ ] Handle attachment deletion and deleted-account placeholders.

Acceptance: two real sessions exchange text/media; unauthorized users cannot read a conversation; concurrent starts create one conversation; retries do not duplicate messages; follow/block changes affect sending immediately.

### Stage 7 — Safety, moderation, and account deletion

- [ ] Implement block/unblock controls and the finalized non-messaging blocking rules.
- [ ] Add reporting flows for posts, comments, and messages.
- [ ] Build the role-restricted report queue and moderation actions with minimal necessary evidence.
- [ ] Enforce suspended-account restrictions on backend reads/writes as defined during implementation.
- [ ] Implement retryable account deletion across Clerk, Convex records, messages, and media.
- [ ] Add necessary privacy/support information and clear deletion confirmation.

Acceptance: regular users cannot perform moderator actions; reports are private; suspension blocks prohibited actions; interrupted deletion resumes and removes all owned content without deleting another user's content.

### Stage 8 — Reliability, costs, and TestFlight readiness

- [ ] Add minimal operational logs and useful error reporting without logging tokens, private message bodies, or protected URLs.
- [ ] Monitor failures, cleanup backlog, storage usage, and delivery usage.
- [ ] Exercise upload-pausing behavior and clear user feedback at capacity.
- [ ] Run backend authorization/invariant tests, lint, and TypeScript checks.
- [ ] Run real-iPhone checks for both sign-in providers, all core flows, media, slow networks, retries, and accessibility.
- [ ] Verify English copy, light/dark themes, safe areas, keyboard avoidance, and media-library denial states.
- [ ] Prepare a release environment and TestFlight build through EAS, with a smoke-test and rollback/rebuild procedure.
- [ ] Invite the initial group only after audience/privacy, provider setup, and release checks pass.

Acceptance: the agreed v1 works end to end on real iPhones, backend authorization tests pass, costs have operational limits, and no unresolved privacy blocker remains.

## 7. Assumptions and proposed defaults

These are distinct from explicitly confirmed requirements. Resolve material differences before implementing the relevant behavior.

| Item | Proposed default / assumption |
| --- | --- |
| Admission | TestFlight distribution controls invitations; no in-app invitation codes or membership approval. TestFlight distribution is not a strong backend membership allowlist. |
| Public browsing | Public means visible to signed-in app members; no anonymous browsing. |
| Moderation owner | The project owner is the initial moderator, with the role provisioned securely rather than self-selected. |
| Accessibility | Accessible labels, readable contrast, scalable text where practical, and adequate touch targets are part of v1. |
| Publishing retries | No partially published posts; retry operations must not duplicate content. |
| Block effects outside DMs | Prevent new follows and social interactions between blocked users, remove follow/request relationships, and suppress discovery between them. Preserve past DM history as explicitly agreed. Finalize treatment of old comments/likes before implementation. |
| Unblocking | Does not automatically restore prior follows or sending permission; mutual follows must be established again. |
| Private-to-public switch | Pending follow requests become accepted follows; validate this default before implementing the transition. |
| Admin placement | Admin review lives behind a restricted entry in Profile/settings, not a fifth main tab. |
| Follow-request placement | A Profile/settings entry and in-app indicator expose a dedicated request-management screen. |
| Video playback | Prefer deliberate playback with only one active video at a time to limit bandwidth and resource use; finalize during media UX work. |
| Environments | Isolate development and TestFlight release data/configuration as allowed by selected service plans; do not promise extra paid environments. |
| Analytics | Start with operational health and minimal aggregate usage, not advertising or behavioral tracking SDKs. |

## 8. Open risks and decisions

1. **Protected media delivery:** Convex's standard `storage.getUrl()` links are bearer-accessible and remain usable until deletion. Do not use those links directly for protected content. Validate an authenticated delivery approach, including video ranges and caches, before building the full media pipeline.
2. **Free-tier capacity:** Short videos can still be large, and repeated viewing consumes bandwidth. Stopping uploads alone does not control download traffic. Exact free-tier allowances and throttling behavior must be checked at implementation time.
3. **SDK compatibility:** Exact Clerk, NativeWind, and Convex package versions remain unselected. Validate against the installed Expo/React Native stack rather than silently upgrading or replacing tools.
4. **Teenage audience:** Age/privacy/consent and moderation requirements need a focused review before real-user release. Do not infer legal compliance from TestFlight distribution or mutual-follow messaging.
5. **Provider and release setup:** Apple/Google provider configuration, membership activation, build credentials, and TestFlight readiness are not yet verified.
6. **Deletion consistency:** Clerk and Convex cleanup is not one database transaction. Design retryable orchestration, temporary access denial, and orphan-file cleanup.
7. **Media validation:** Library metadata and client validation are not a sufficient security boundary. Determine feasible backend verification and format handling without assuming a transcoding service.
8. **Blocking semantics:** Non-messaging effects and historical social interactions need resolution using the defaults above.
9. **Budget boundaries:** The free-tier objective is not a guarantee of unlimited availability. Define the response to read/delivery capacity exhaustion as well as upload limits.

## 9. Test and release checklist

- [ ] Google and Apple sign-in, sign-out, session restoration, and incomplete onboarding.
- [ ] Unique username conflicts and owner-only profile edits.
- [ ] Private/public transitions, request approval/rejection, follower removal, and direct access checks.
- [ ] Mixed-media limits, denied library access, oversized/unsupported media, partial failures, retries, and cleanup.
- [ ] Feed order, pagination, empty states, public-only Explore, and people search.
- [ ] Like deduplication, comment ownership, deleted posts, and revoked visibility.
- [ ] Mutual-follow messaging, unread indicators, concurrent conversation creation, duplicate-send prevention, and attachment authorization.
- [ ] Unfollow/block stops new messages while preserving existing authorized history.
- [ ] Moderator-only operations, report confidentiality, and suspension enforcement.
- [ ] Account deletion removes owned content and retries partial failures safely.
- [ ] Copied protected-media links, revoked post access, and cache behavior do not bypass authorization.
- [ ] Capacity controls, actionable failure feedback, and observability without sensitive logs.
- [ ] Real-iPhone usability, light/dark mode, accessibility, scrolling, video resources, and keyboard behavior.
- [ ] `npx expo lint` and `npx tsc --noEmit` pass for implementation work.

## 10. Future compatibility without building future features

- Keep shared business logic cross-platform so Android can follow without a backend rewrite.
- Keep conversations and membership explicit so group chats can be designed later, while enforcing exactly two participants in v1.
- Keep media lifecycle handling reusable for future stories, without adding story expiration or story UI now.
- Avoid iOS-only assumptions outside platform-specific integration boundaries.

## 11. Documentation references

Recheck these sources and exact package versions when implementation begins.

- Expo SDK 58: https://docs.expo.dev/versions/v58.0.0/
- Expo documentation index: https://docs.expo.dev/llms.txt
- Expo Router: https://docs.expo.dev/router/introduction/
- Expo/EAS: https://docs.expo.dev/eas/
- Convex file storage and URL security: https://docs.convex.dev/file-storage/overview
- Convex pricing and capacity: https://www.convex.dev/pricing

The Expo SDK/index and Convex storage/pricing pages were consulted during the interview. Other integration details still require version-specific verification.
