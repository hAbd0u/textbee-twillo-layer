# Graph Report - textbee  (2026-11-07)

## Corpus Check
- 673 files · ~450,759 words
- Verdict: corpus is large enough that graph structure adds value.
- Unclassified: 58 file(s) not represented in the graph (top: .xml 26, (none) 12, .hbs 8)

## Summary
- 4674 nodes · 12705 edges · 179 communities (149 shown, 30 thin omitted)
- Extraction: 98% EXTRACTED · 2% INFERRED · 0% AMBIGUOUS · INFERRED: 307 edges (avg confidence: 0.83)
- Token cost: 0 input · 0 output

## Graph Freshness
- Built from commit: `8f4b5f82`
- Run `git rev-parse HEAD` and compare to check if the graph is stale.
- Run `graphify update .` after code changes (no API cost).

## Community Hubs (Navigation)
- DashboardScreen.kt
- @nestjs/common
- cn
- auth.controller.ts
- device-list.tsx
- .getSharedPreferenceString
- billing.service.ts
- mongoose
- create-webhook-dialog.tsx
- OnboardingViewModel.kt
- WebhookService
- use-bulk-send.ts
- mail.service.ts
- utils.ts
- MainActivity.java
- AuthService
- error-history.ts
- analytics/attribution.ts
- gateway.dto.ts
- NewMainActivity.kt
- support.controller.ts
- TextbeeUtils.java
- [planName]/page.tsx
- MessagesScreen.kt
- lucide-react
- react
- use-toast.ts
- gateway.service.ts
- BootCompletedReceiver.kt
- api/package.json
- vitest
- GatewayController
- dependencies
- build-public-document.ts
- auth.ts
- web/package.json
- next
- mock-api.ts
- AnalyticsService
- update-app-helpers.ts
- .build
- GatewayService
- message-history/index.tsx
- dependencies
- ApiManager.java
- condition-evaluator.ts
- hooks.ts
- SMS
- email-templates.service.ts
- notifications.controller.ts
- webhook-docs.tsx
- BillingService
- email-render.ts
- dashboard-notification.schema.ts
- plan-picker.tsx
- sms-details-dialog.tsx
- useSubscription
- fixtures.ts
- devDependencies
- SendSlotSchedulerTest
- polar-subscription-sync.spec.ts
- user.schema.ts
- device-filter.test.tsx
- BillingController
- message-query.ts
- conformance.spec.ts
- failure-view.ts
- test
- UserRollupService
- subscription-info.tsx
- SMSDTO
- devDependencies
- User Meta Population Optimization Guide
- preview-emails.ts
- NotificationContextLoader
- SMSFilterHelperTest
- sms-permission-status.ts
- NotificationsService
- compilerOptions
- data-table.tsx
- notification-provider.tsx
- SettingsChangeWatcher
- SimReport
- EmailController
- usage-summary.tsx
- compilerOptions
- What's Done
- access-footprint.service.ts
- os-version.ts
- mail/email-links.ts
- pending-recovery.ts
- OnboardingViewModel
- .updateSMSStatus
- notification-ranker.ts
- SmsDedupeStore
- SMSBroadcastReceiver.kt
- README.md
- lib/email-links.ts
- route-tabs.tsx
- SimLookup
- SMSFilterRule
- scripts
- .canPerformAction
- use-onboarding.ts
- relative-time.tsx
- components.json
- SmsSendWorker
- access-footprint.ts
- RegistrationChecks
- SMSFilterViewModel
- billing.controller.ts
- user-agent.ts
- SesEventsService
- sim-label.ts
- .evaluate
- SMSFilterHelper
- search-registry.test.ts
- Android Build Variants Setup (Dev vs Prod)
- fcm-push-options.ts
- FCMService
- .doWork
- .handleSendSms
- scripts
- ExampleInstrumentedTest.java
- PermissionFailureNotifierTest
- .receiveSMS
- .refresh
- breadcrumbs.ts
- billing.ts
- StickyNotificationService
- SmsDedupeStoreTest
- jest
- batch-summary.ts
- .shouldRetry
- FakeProbe
- dependencies
- SubscriptionStatus
- tsconfig.build.json
- translate-safe-dom.test.tsx
- device-list.test.tsx
- webhooks-section.test.tsx
- ErrorBoundary.tsx
- support-hq-widget.test.tsx
- nest-cli.json
- SECURITY.md
- subscription-info.test.tsx
- app-header.test.tsx
- .shouldPoll
- 1. Firebase Configuration
- gradlew
- checkoutSource
- recipients.ts
- Self-hosting
- Testing
- .shouldNotify
- HeartbeatResponseDTOTest
- sent
- device-selection.test.ts
- sim-id-input.test.tsx
- ResizeObserverMock
- HeartbeatResponseDTO
- LoggerMiddleware
- next.config.js
- eslint-config-next
- next-env.d.ts
- no-third-party.spec.ts
- SMSForwardResponseDTO.kt
- GEMINI.md
- Contributing
- third-party-hosts.ts
- HeartbeatInputDTO

## God Nodes (most connected - your core abstractions)
1. `cn()` - 195 edges
2. `react` - 117 edges
3. `lucide-react` - 85 edges
4. `@nestjs/common` - 73 edges
5. `mongoose` - 72 edges
6. `Button` - 69 edges
7. `User` - 66 edges
8. `vitest` - 63 edges
9. `@nestjs/mongoose` - 62 edges
10. `next` - 56 edges

## Surprising Connections (you probably didn't know these)
- `Phase 6 — Background Services & Receivers ✅ Complete` --references--> `errorMessage()`  [INFERRED]
  android/MIGRATION.md → web/app/(app)/dashboard/(components)/devices/device-list.tsx
- `Data Layer — Kotlin Stubs (Phase 4) ✅` --references--> `HeartbeatHelper`  [INFERRED]
  android/MIGRATION.md → android/app/src/main/java/com/vernu/sms/helpers/HeartbeatHelper.kt
- `Phase 3 — SMS Filter Screen ✅ Complete` --references--> `NewMainActivity`  [INFERRED]
  android/MIGRATION.md → android/app/src/main/java/com/vernu/sms/ui/main/NewMainActivity.kt
- `Theme` --references--> `TextbeeTheme()`  [INFERRED]
  android/MIGRATION.md → android/app/src/main/java/com/vernu/sms/ui/theme/Theme.kt
- `Architecture` --references--> `sent()`  [INFERRED]
  README.md → web/app/(app)/dashboard/(components)/message-history/message-row.test.tsx

## Import Cycles
- 3-file cycle: `api/src/billing/billing.module.ts -> api/src/gateway/gateway.module.ts -> api/src/webhook/webhook.module.ts -> api/src/billing/billing.module.ts`

## Communities (179 total, 30 thin omitted)

### Community 0 - "DashboardScreen.kt"
Cohesion: 0.04
Nodes (100): activitycompat, activitynotfoundexception, activityresultcontracts, addpathnodes, alignment, Activity, Context, Intent (+92 more)

### Community 1 - "@nestjs/common"
Cohesion: 0.03
Nodes (85): AuthModule, Module, API_KEY_PREFIX, AuthGuard, Injectable, OptionalAuthGuard, Injectable, ApiKeyTombstone (+77 more)

### Community 2 - "cn"
Cohesion: 0.04
Nodes (89): class-variance-authority, cmdk, @radix-ui/react-alert-dialog, @radix-ui/react-avatar, @radix-ui/react-dialog, @radix-ui/react-icons, @radix-ui/react-navigation-menu, @radix-ui/react-scroll-area (+81 more)

### Community 3 - "auth.controller.ts"
Cohesion: 0.06
Nodes (72): API_KEY_ID_PARAM, AuthController, ApiBearerAuth, ApiOperation, ApiParam, ApiQuery, ApiResponse, ApiSecurity (+64 more)

### Community 4 - "device-list.tsx"
Cohesion: 0.06
Nodes (70): react-qr-code, ApiKeyRow, ApiKeys(), GenerateApiKey, GenerateApiKeyHandle, GenerateApiKeyProps, AddDeviceDialog, AddDeviceHandle (+62 more)

### Community 5 - ".getSharedPreferenceString"
Cohesion: 0.05
Nodes (29): DeviceConfigDTO, SimInfoCollectionDTO, RegisterDeviceInputDTO, SimInfoDTO, SimInfoCollectionDTO, DeviceConfig, Context, DeviceConfigDTO (+21 more)

### Community 6 - "billing.service.ts"
Cohesion: 0.04
Nodes (65): BillingNotificationsService, api_src_billing_billing_notifications_service_billingnotificationtype, emailAttemptCovers(), FAILED_EMAIL_RETRY_MS, NOTICE_REFRESH_MS, noticePeriodStart(), NotifyOnceInput, Injectable (+57 more)

### Community 7 - "mongoose"
Cohesion: 0.06
Nodes (60): BackfillModule, ONLY_MISSING, Module, WRITE, AccessFootprintSchema, ApiKey, ApiKeySchema, Prop (+52 more)

### Community 8 - "create-webhook-dialog.tsx"
Cohesion: 0.06
Nodes (71): @hookform/resolvers, react-hook-form, uuid, zod, authButtonClass, authFieldClass, LoginForm(), LoginFormValues (+63 more)

### Community 9 - "OnboardingViewModel.kt"
Cohesion: 0.05
Nodes (44): ApiManagerKt, SimInfoDTO, serverErrorMessage(), StateFlow, DashboardState, DashboardViewModel, AndroidViewModel, StateFlow (+36 more)

### Community 10 - "WebhookService"
Cohesion: 0.06
Nodes (42): Process, Processor, WebhookQueueProcessor, ApiBearerAuth, ApiOperation, ApiParam, ApiQuery, ApiResponse (+34 more)

### Community 11 - "use-bulk-send.ts"
Cohesion: 0.06
Nodes (54): papaparse, buildRecipientPlan(), CsvRow, detectRecipientColumn(), EXACT_ONLY_HINTS, ExcludedRow, ExclusionReason, extractTemplateVariables() (+46 more)

### Community 12 - "mail.service.ts"
Cohesion: 0.05
Nodes (41): Cron, Injectable, InjectModel, VerificationReminderTask, EligibilityUser, EmailCategory, ESSENTIAL_TEMPLATES, skipReason (+33 more)

### Community 13 - "utils.ts"
Cohesion: 0.07
Nodes (46): @radix-ui/react-label, @radix-ui/react-switch, react-syntax-highlighter, MapStep(), ReviewStep(), StepShell(), BulkSendState, deviceName() (+38 more)

### Community 14 - "MainActivity.java"
Cohesion: 0.06
Nodes (46): Adapter, alertdialog, MainActivity, FilterRulesAdapter, SMSFilterActivity, ViewHolder, AppConstants, android.content.Intent (+38 more)

### Community 15 - "AuthService"
Cohesion: 0.05
Nodes (22): AuthService, build(), Injectable, InjectModel, withoutPassword(), CanModifyApiKey, Injectable, JwtStrategy (+14 more)

### Community 16 - "error-history.ts"
Cohesion: 0.07
Nodes (49): capErrorHistory(), cappedTrail(), currentFailureEntry(), DISPATCH_FAILED_ERROR_CODE, ERROR_HISTORY_FIRST, ERROR_HISTORY_LIMIT, ERROR_HISTORY_NEWEST, errorHistoryAppend() (+41 more)

### Community 17 - "analytics/attribution.ts"
Cohesion: 0.07
Nodes (49): ActionButton(), isExternal(), Analytics(), freshRender(), AttributionCapture(), Attribution, ATTRIBUTION_COOKIE, ATTRIBUTION_MAX_AGE_DAYS (+41 more)

### Community 18 - "gateway.dto.ts"
Cohesion: 0.08
Nodes (55): ACCOUNT_BLOCKED_RESPONSE, DEVICE_ID_PARAM, DEVICE_NOT_FOUND_RESPONSE, INVALID_DEVICE_ID_RESPONSE, UNAUTHORIZED_RESPONSE, AppVersionInfoDTO, BatteryInfoDTO, CursorPaginationMetaDTO (+47 more)

### Community 19 - "NewMainActivity.kt"
Cohesion: 0.05
Nodes (45): activity, Context, RecoveryPoll, Bundle, ComponentActivity, NavHostController, MainDestination, DASHBOARD (+37 more)

### Community 20 - "support.controller.ts"
Cohesion: 0.06
Nodes (41): JwtAuthGuard, Injectable, Injectable, TurnstileService, firstName(), CreateSupportMessageDto, SupportCategory, ACCOUNT_DELETION (+33 more)

### Community 21 - "TextbeeUtils.java"
Cohesion: 0.08
Nodes (37): HeartbeatInputDTO, SimInfoCollectionDTO, Context, HeartbeatWorker, Worker, android.telephony.SubscriptionInfo, batterymanager, build (+29 more)

### Community 22 - "[planName]/page.tsx"
Cohesion: 0.07
Nodes (24): @tanstack/react-query, PlanChangePreview, EditProfileForm(), TODO: Implement email verification, UpdateProfileFormData, updateProfileSchema, StepActions(), StepActionsProps (+16 more)

### Community 23 - "MessagesScreen.kt"
Cohesion: 0.08
Nodes (31): android, SmsMessage, DeviceLog, Context, LogEntry, ActivityTimeline, SmsMessage, TimelineStep (+23 more)

### Community 24 - "lucide-react"
Cohesion: 0.11
Nodes (29): lucide-react, CommunitySupportCard(), SmsPermissionAlert(), encouragement(), GetStartedCard(), GetStartedCardSkeleton(), Overview(), SendSms() (+21 more)

### Community 25 - "react"
Cohesion: 0.06
Nodes (35): next-themes, react, isNavItemActive(), mobileNavItems, NavItem, navItems, CommandMenu(), DashboardLayout() (+27 more)

### Community 26 - "use-toast.ts"
Cohesion: 0.06
Nodes (45): @radix-ui/react-toast, ContributePage(), ChangePasswordForm(), ChangePasswordFormData, changePasswordSchema, DeleteWebhookButton(), EditWebhookDialog(), WebhookCard() (+37 more)

### Community 27 - "gateway.service.ts"
Cohesion: 0.06
Nodes (26): escapeRegExp(), encodeCursor(), literalSetStage(), LeanSMS, TODO: Implement a queue to send the SMS if recipients are too many, TODO: add more tests for different app version codes, STATUS_EVENTS, DEFAULT_BULK_DISPATCH_MAX_SPREAD_HOURS (+18 more)

### Community 28 - "BootCompletedReceiver.kt"
Cohesion: 0.07
Nodes (28): GatewayStatsData, GatewayStatsResponse, MessagesResponse, PaginationMeta, SendSmsRequest, RegisterDeviceResponseDTO, SubscriptionPlan, SubscriptionResponse (+20 more)

### Community 29 - "api/package.json"
Cohesion: 0.04
Nodes (44): author, description, engines, node, axios, eslint, @types/node, typescript (+36 more)

### Community 30 - "vitest"
Cohesion: 0.11
Nodes (23): msw, @testing-library/react, vitest, getSessionSpy, web_components_notifications_dismissal_migration_migration_marker, renderStack(), OLD_FAILURE, getSessionMock (+15 more)

### Community 31 - "GatewayController"
Cohesion: 0.19
Nodes (21): GatewayController, parsePagination(), ApiBearerAuth, ApiOperation, ApiParam, ApiQuery, ApiResponse, ApiSecurity (+13 more)

### Community 32 - "dependencies"
Cohesion: 0.05
Nodes (44): dependencies, axios, class-variance-authority, clsx, cmdk, date-fns, @hookform/resolvers, lucide-react (+36 more)

### Community 33 - "build-public-document.ts"
Cohesion: 0.09
Nodes (36): AppModule, Module, bootstrap(), logger, assertNoDeprecatedOperations(), collectSchemaRefs(), createPublicOpenApiDocument(), eachOperation() (+28 more)

### Community 34 - "auth.ts"
Cohesion: 0.08
Nodes (30): APP_REFERRERS, AttributionEntryInput, AttributionTouchInput, cleanString(), cleanTimestamp(), normalizeReferrer(), normalizeSignupSource(), REFERRER_SOURCES (+22 more)

### Community 35 - "web/package.json"
Cohesion: 0.05
Nodes (38): clsx, @eslint/eslintrc, jsdom, postcss, @radix-ui/react-checkbox, @radix-ui/react-collapsible, @radix-ui/react-dropdown-menu, @radix-ui/react-popover (+30 more)

### Community 36 - "next"
Cohesion: 0.08
Nodes (9): next, next-auth, AuthShell(), AuthShellProps, LoginWithGoogle(), Footer(), links, Routes (+1 more)

### Community 37 - "mock-api.ts"
Cohesion: 0.16
Nodes (12): page(), @playwright/test, CSV_WITH_PROBLEM_ROWS, AUTHED_PAGES, json(), mockApi(), MockApiOverrides, authenticate() (+4 more)

### Community 38 - "AnalyticsService"
Cohesion: 0.10
Nodes (18): AnalyticsModule, ConsumerModule, ConsumerProbe, Injectable, Module, Module, AnalyticsService, AnalyticsUser (+10 more)

### Community 39 - "update-app-helpers.ts"
Cohesion: 0.09
Nodes (31): DeviceList(), DeviceTags(), errorMessage(), reportsSmsPermissionOff(), devicePresence, HintInput, needsNewerAppForPresence(), onlineWindowMs() (+23 more)

### Community 40 - ".build"
Cohesion: 0.10
Nodes (17): DeviceHealthRows, HealthAction, GRANT_NOTIFICATIONS, GRANT_SMS, NONE, OPEN_APP_SETTINGS, OPEN_BATTERY_SETTINGS, SEND_HEARTBEAT (+9 more)

### Community 41 - "GatewayService"
Cohesion: 0.10
Nodes (12): appVersionMetadata, VersionedDevice, GatewayService, Injectable, userIdOf(), CanModifyDevice, Injectable, DispatchPlan (+4 more)

### Community 42 - "message-history/index.tsx"
Cohesion: 0.13
Nodes (27): ActiveFilters(), ActiveFiltersProps, countExtraFilters(), DEFAULT_EXTRA_FILTERS, formatDay(), isBatchId(), isDay(), isStatus() (+19 more)

### Community 43 - "dependencies"
Cohesion: 0.06
Nodes (34): dependencies, axios, bcryptjs, bowser, bull, class-validator, dotenv, express (+26 more)

### Community 44 - "ApiManager.java"
Cohesion: 0.12
Nodes (21): ApiManager, GatewayApiService, body, com.vernu.sms.dtos.HeartbeatInputDTO, com.vernu.sms.dtos.HeartbeatResponseDTO, com.vernu.sms.dtos.PendingMessagesResponseDTO, com.vernu.sms.dtos.RegisterDeviceInputDTO, com.vernu.sms.dtos.RegisterDeviceResponseDTO (+13 more)

### Community 45 - "condition-evaluator.ts"
Cohesion: 0.10
Nodes (28): ATTRIBUTE_GROUPS, ATTRIBUTES, ATTRIBUTES_BY_KEY, FLAG, LIST, NUMERIC, TEXT, and() (+20 more)

### Community 46 - "hooks.ts"
Cohesion: 0.08
Nodes (26): ChangePasswordPayload, DeviceMessagesEnvelope, DeviceMessagesParams, ListEnvelope, ListQueryOpts, MutationOpts, QueryOpts, SendSmsPayload (+18 more)

### Community 47 - "SMS"
Cohesion: 0.11
Nodes (22): errorHistoryEntry, InjectModel, InjectModel, DeviceTombstoneDocument, SMSBatch, Prop, Schema, SMS (+14 more)

### Community 48 - "email-templates.service.ts"
Cohesion: 0.10
Nodes (24): placeholders(), RenderedDocument, allowedPlaceholders(), ALWAYS_ENABLED, baseCopy(), CACHE_TTL_MS, Copy, COPY_FIELDS (+16 more)

### Community 49 - "notifications.controller.ts"
Cohesion: 0.12
Nodes (25): EVENT_TYPES, NotificationsController, ApiBearerAuth, ApiOperation, ApiResponse, ApiTags, Body, Controller (+17 more)

### Community 50 - "webhook-docs.tsx"
Cohesion: 0.10
Nodes (24): @radix-ui/react-accordion, @radix-ui/react-tabs, message_received_payload, message_send_template, sms_delivered_payload, sms_sent_failed, sms_sent_payload, WebhookDocs() (+16 more)

### Community 51 - "BillingService"
Cohesion: 0.12
Nodes (4): BillingService, defined(), toDate(), Injectable

### Community 52 - "email-render.ts"
Cohesion: 0.15
Nodes (27): applySections(), Block, escapeHtml(), inlineHtml(), inlineText(), isLower(), isUpper(), linkText() (+19 more)

### Community 53 - "dashboard-notification.schema.ts"
Cohesion: 0.12
Nodes (27): DismissMode, NotificationKind, Placement, DashboardNotification, DISMISS_MODES, NOTIFICATION_KINDS, NOTIFICATION_PLACEMENTS, NOTIFICATION_STATUSES (+19 more)

### Community 54 - "plan-picker.tsx"
Cohesion: 0.20
Nodes (25): CheckoutPage(), formatPlan(), YearlySwitchCard(), PlanCard(), PlanPicker(), PlanPickerProps, useBillingPlans(), BillingInterval (+17 more)

### Community 55 - "sms-details-dialog.tsx"
Cohesion: 0.16
Nodes (16): FAILED, OFFICE, dayLabel(), groupMessagesByDay(), messageDate(), MessageDay, MessageRow(), MessageRowProps (+8 more)

### Community 56 - "useSubscription"
Cohesion: 0.16
Nodes (21): DeleteAccountForm(), AccountDeletionAlert(), JoinDiscordBanner(), PastDueBillingAlert(), envDiscountCode, envDiscountPercentage, UpgradeToProAlert(), VerifyEmailAlert() (+13 more)

### Community 57 - "fixtures.ts"
Cohesion: 0.12
Nodes (19): monthlyPro, post, useBillingPlans, useBillingPlans, useSubscription, mockApiKeys, mockBillingPlans, mockFreeSubscription (+11 more)

### Community 58 - "devDependencies"
Cohesion: 0.07
Nodes (27): devDependencies, eslint, eslint-config-next, @eslint/eslintrc, jsdom, msw, @playwright/test, postcss (+19 more)

### Community 59 - "SendSlotSchedulerTest"
Cohesion: 0.14
Nodes (8): Keys, Context, PrefsSlotStore, SendSlotScheduler, Slot, SlotStore, MemoryStore, SendSlotSchedulerTest

### Community 60 - "polar-subscription-sync.spec.ts"
Cohesion: 0.14
Nodes (16): PolarSubscriptionSync, decideSync(), ENDED_STATUSES, LinkedRow, PolarSnapshot, REPLACED_STATUS, snapshotHasEnded(), apply() (+8 more)

### Community 61 - "user.schema.ts"
Cohesion: 0.14
Nodes (24): Attribution, AttributionEntry, AttributionEntrySchema, AttributionSchema, AttributionTouch, AttributionTouchSchema, ClientInfo, ClientInfoSchema (+16 more)

### Community 62 - "device-filter.test.tsx"
Cohesion: 0.08
Nodes (13): @testing-library/user-event, push, refresh, signIn, push, signIn, post, post (+5 more)

### Community 63 - "BillingController"
Cohesion: 0.20
Nodes (16): BillingController, ApiBearerAuth, ApiOperation, ApiQuery, ApiResponse, ApiSecurity, ApiTags, Body (+8 more)

### Community 64 - "message-query.ts"
Cohesion: 0.17
Nodes (18): decodeCursor(), MessageCursor, MessageDirection, parseDirectionInput(), toDirection(), toStoredType(), asSingleString(), badRequest() (+10 more)

### Community 65 - "conformance.spec.ts"
Cohesion: 0.15
Nodes (21): CONDITION_CASES, ConditionCase, HREF_CASES, HrefCase, SELECTION_CASES, SELECTION_DEFAULT_NOW, SelectionCase, ALLOWED (+13 more)

### Community 66 - "failure-view.ts"
Cohesion: 0.20
Nodes (21): date-fns, FailureDetails(), Timeline(), currentIndex(), distinct(), entryView, ERROR_HISTORY_LIMIT, errorMeta() (+13 more)

### Community 67 - "test"
Cohesion: 0.21
Nodes (9): assertequals, assertfalse, assertnotequals, assertnotnull, assertnull, asserttrue, decision, gson (+1 more)

### Community 68 - "UserRollupService"
Cohesion: 0.12
Nodes (10): Cron, Injectable, UserRollupTask, build(), chain(), USER_ID, Injectable, InjectModel (+2 more)

### Community 69 - "subscription-info.tsx"
Cohesion: 0.19
Nodes (17): DateTile(), LimitTile(), LimitTileProps, Meter, StatusPill(), SubscriptionInfo(), SubscriptionStatus, formatDate() (+9 more)

### Community 70 - "SMSDTO"
Cohesion: 0.18
Nodes (12): SimUsedDTO, SMSDTO, BroadcastReceiver, Context, Intent, SMSStatusReceiver, Context, Result (+4 more)

### Community 71 - "devDependencies"
Cohesion: 0.09
Nodes (23): devDependencies, eslint, eslint-config-prettier, eslint-plugin-prettier, inline-css, jest, @nestjs/cli, @nestjs/schematics (+15 more)

### Community 72 - "User Meta Population Optimization Guide"
Cohesion: 0.09
Nodes (22): 1. MongoDB Aggregation Pipeline, 1. N+1 Query Problem, 2. Bulk SMS Count Calculation, 2. Sequential Processing, 3. Batch Processing, 3. Redundant Calculations, 4. Bulk Write Operations, 4. Memory Usage (+14 more)

### Community 73 - "preview-emails.ts"
Cohesion: 0.13
Nodes (18): main(), OUT_DIR, SAMPLES, validate(), YEAR, f, compiled, getEnv() (+10 more)

### Community 74 - "NotificationContextLoader"
Cohesion: 0.17
Nodes (12): daysSince(), daysUntil(), groupOf(), NotificationContextLoader, account(), build(), chain(), NOW (+4 more)

### Community 76 - "sms-permission-status.ts"
Cohesion: 0.16
Nodes (18): deviceLabel(), loadSmsPermissionStatus(), reportsSmsPermissionOff(), SmsPermissionStatus, chain(), DEVICE_ID, FAILED_AT, models() (+10 more)

### Community 77 - "NotificationsService"
Cohesion: 0.13
Nodes (11): NotificationsService, build(), chain(), NOTIFICATION_ID, OTHER_USER_ID, USER_ID, Injectable, InjectModel (+3 more)

### Community 78 - "compilerOptions"
Cohesion: 0.09
Nodes (21): compilerOptions, allowSyntheticDefaultImports, declaration, emitDecoratorMetadata, experimentalDecorators, forceConsistentCasingInFileNames, incremental, module (+13 more)

### Community 79 - "data-table.tsx"
Cohesion: 0.14
Nodes (19): @tanstack/react-table, DataTable(), DataTableProps, WebhookTableFeatures, buildDeviceLabel(), columns, formatDate(), ProductColumns (+11 more)

### Community 80 - "notification-provider.tsx"
Cohesion: 0.15
Nodes (20): alreadyMigrated(), LEGACY_DISMISSAL_KEYS, LEGACY_DISMISSALS, markMigrated(), pendingDismissalEvents(), read(), NotificationContext, NotificationContextValue (+12 more)

### Community 81 - "SettingsChangeWatcher"
Cohesion: 0.16
Nodes (10): ActivityLifecycleCallbacks, android.app.Application, Activity, Bundle, SettingsChangeWatcher, SMSGatewayApplication, androidx.work.Configuration, executors (+2 more)

### Community 82 - "SimReport"
Cohesion: 0.17
Nodes (4): SimUsedDTO, SimUsedDTO, SimReport, SimReportTest

### Community 83 - "EmailController"
Cohesion: 0.20
Nodes (14): EmailController, sendPage(), ApiExcludeController, Body, Controller, Get, Post, Query (+6 more)

### Community 84 - "usage-summary.tsx"
Cohesion: 0.15
Nodes (9): UsageSummary(), UsageTile(), Skeleton(), Subscription, buildWindow(), deriveUsage(), NEAR_LIMIT_PERCENT, UNLIMITED (+1 more)

### Community 85 - "compilerOptions"
Cohesion: 0.10
Nodes (20): compilerOptions, allowJs, baseUrl, esModuleInterop, forceConsistentCasingInFileNames, incremental, isolatedModules, jsx (+12 more)

### Community 86 - "What's Done"
Cohesion: 0.10
Nodes (19): Activities (Legacy UI), Android Kotlin/Compose Migration, Core App, Data Layer — Kotlin Stubs (Phase 4) ✅, Helpers, Key Constraints to Keep in Mind, Migration Roadmap, Overview (+11 more)

### Community 87 - "access-footprint.service.ts"
Cohesion: 0.15
Nodes (11): AccessChannel, AccessFootprintService, FOOTPRINT_PLACE_FIELDS, headerValue(), ORIGINAL_ENV, Injectable, InjectModel, AccessFootprint (+3 more)

### Community 88 - "os-version.ts"
Cohesion: 0.17
Nodes (10): RegisterDeviceInputDTO, BUILD_ID_RELEASE, deriveOsVersion(), isSource(), normalizeOsFields(), OsFieldsInput, OsVersionSource, parseReleaseFromBuildId() (+2 more)

### Community 89 - "mail/email-links.ts"
Cohesion: 0.24
Nodes (14): apiPublicUrl(), appPublicUrl(), base(), billingEmailUrl(), billingUrl(), defaultBillingInterval(), emailLinkSecret(), emailSource() (+6 more)

### Community 90 - "pending-recovery.ts"
Cohesion: 0.16
Nodes (14): DeviceConfig, deviceConfigFor(), envBool(), envInt(), fleetDeviceConfig(), OVERRIDABLE_KEYS, RECOVERY_MIN_VERSION_CODE, CLAIM_LIMIT (+6 more)

### Community 91 - "OnboardingViewModel"
Cohesion: 0.12
Nodes (11): NavHostController, OnboardingNavGraph(), StateFlow, OnboardingState, OnboardingViewModel, CredentialsScreen(), ManualTab(), QrTab() (+3 more)

### Community 92 - ".updateSMSStatus"
Cohesion: 0.19
Nodes (14): notRecordedYet(), hasNestedSimUsed(), resolveSimSelection(), resolveSimUsed(), resolveSimUsedReport(), SIM_SELECTIONS, SimSelection, SimUsedReport (+6 more)

### Community 93 - "notification-ranker.ts"
Cohesion: 0.19
Nodes (18): accountReason(), asTime(), Block, daysBetween(), DismissRule, FilteredNotification, FrequencyRule, pickVariant() (+10 more)

### Community 94 - "SmsDedupeStore"
Cohesion: 0.20
Nodes (4): DedupeStorage, Context, SharedPreferencesStorage, SmsDedupeStore

### Community 95 - "SMSBroadcastReceiver.kt"
Cohesion: 0.16
Nodes (11): BroadcastReceiver, Context, Intent, SMSBroadcastReceiver, Context, Result, Worker, SMSReceivedWorker (+3 more)

### Community 96 - "README.md"
Cohesion: 0.11
Nodes (17): Community and support, FAQ, License, Quick start, Receive SMS, Screenshots, SDKs and integrations, Security (+9 more)

### Community 97 - "lib/email-links.ts"
Cohesion: 0.20
Nodes (12): dynamic, GET(), dynamic, GET(), dynamic, GET(), emailLinkApiUrl(), emailLinkRedirect() (+4 more)

### Community 98 - "route-tabs.tsx"
Cohesion: 0.18
Nodes (7): CommunityLinks(), PageHeader(), PageHeaderProps, isTabActive(), RouteTab, RouteTabs(), tabs

### Community 99 - "SimLookup"
Cohesion: 0.27
Nodes (5): Context, Intent, SimUsedDTO, SubscriptionInfo, SimLookup

### Community 100 - "SMSFilterRule"
Cohesion: 0.14
Nodes (14): FilterTarget, BOTH, MESSAGE, SENDER, MatchType, CONTAINS, ENDS_WITH, EXACT (+6 more)

### Community 101 - "scripts"
Cohesion: 0.12
Nodes (17): scripts, backfill:rollups, build, export:openapi, export:sms-errors, format, lint, prebuild (+9 more)

### Community 102 - ".canPerformAction"
Cohesion: 0.19
Nodes (10): formatDate(), formatDateTime(), allowancePercent(), billingPeriod, dailyWindowStart(), DateInput, monthsAfter(), periodAnchor() (+2 more)

### Community 103 - "use-onboarding.ts"
Cohesion: 0.19
Nodes (13): computeStepStates(), StatsShape, StepDef, STEPS, StepState, SubShape, UserShape, OnboardingStatus (+5 more)

### Community 104 - "relative-time.tsx"
Cohesion: 0.21
Nodes (11): SimChipProps, COMPACT_UNITS, RelativeTime(), NOW, toCompactRelativeLabel(), toExactLabel(), toRelativeLabel(), web_components_ui_tooltip_tooltip (+3 more)

### Community 105 - "components.json"
Cohesion: 0.12
Nodes (16): aliases, components, hooks, lib, ui, utils, rsc, $schema (+8 more)

### Community 106 - "SmsSendWorker"
Cohesion: 0.23
Nodes (6): Context, Worker, SmsSendWorker, Data, ForegroundInfo, ListenableFuture

### Community 107 - "access-footprint.ts"
Cohesion: 0.18
Nodes (7): ACCESS_CHANNELS, classifyChannel(), classifyClient(), CLIENT_FAMILIES, RecentKeys, requestPath(), USER_AGENT_FAMILIES

### Community 108 - "RegistrationChecks"
Cohesion: 0.21
Nodes (5): Context, RegistrationChecks, RegistrationChecksTest, connectionresult, googleapiavailability

### Community 109 - "SMSFilterViewModel"
Cohesion: 0.22
Nodes (4): AndroidViewModel, Context, StateFlow, SMSFilterViewModel

### Community 110 - "billing.controller.ts"
Cohesion: 0.29
Nodes (12): UNAUTHORIZED_RESPONSE, BillingNotificationDTO, ChangePlanInputDTO, ChangePlanResponseDTO, CheckoutInputDTO, CheckoutResponseDTO, CurrentSubscriptionResponseDTO, PlanChangePreviewDTO (+4 more)

### Community 111 - "user-agent.ts"
Cohesion: 0.23
Nodes (13): BROWSER_FAMILIES, classifyDevice(), clean(), ClientDescription, describeClient(), DESKTOP_OS, deviceFor(), OS_FAMILIES (+5 more)

### Community 112 - "SesEventsService"
Cohesion: 0.22
Nodes (7): InjectModel, bareAddress(), isSnsUrl(), SesEventsService, snsStringToSign(), Injectable, InjectModel

### Community 113 - "sim-label.ts"
Cohesion: 0.36
Nodes (12): messageDirection(), SimChip(), isNumber(), simCarrier(), simDetails(), simFullLabel(), simLabel(), simMismatch() (+4 more)

### Community 114 - ".evaluate"
Cohesion: 0.22
Nodes (6): AndroidProbe, DeviceHealth, DeviceHealthProbe, DeviceHealthSnapshot, Context, HeartbeatSettingsKeyTest

### Community 115 - "SMSFilterHelper"
Cohesion: 0.21
Nodes (11): Decision, BLOCKED_BY_RULES, BLOCKED_BY_SIM, PROCESS, FilterConfig, FilterMode, ALLOW_LIST, BLOCK_LIST (+3 more)

### Community 116 - "search-registry.test.ts"
Cohesion: 0.16
Nodes (10): ref_node_fs, ref_node_path, @vitejs/plugin-react, searchEntries, SearchEntry, SearchGroup, searchGroupOrder, collectRoutes() (+2 more)

### Community 117 - "Android Build Variants Setup (Dev vs Prod)"
Cohesion: 0.15
Nodes (12): Android Build Variants Setup (Dev vs Prod), Build Errors:, Building the App, Environment Detection, Firebase Issues:, Important Notes, Installation, Key Differences (+4 more)

### Community 118 - "fcm-push-options.ts"
Cohesion: 0.27
Nodes (10): DEFAULT_FCM_HEARTBEAT_TTL_SECONDS, DEFAULT_FCM_SMS_TTL_SECONDS, FCM_MAX_TTL_SECONDS, fcmHeartbeatTtlSeconds(), fcmSmsTtlSeconds(), heartbeatAndroidConfig(), readTtlSeconds(), smsAndroidConfig() (+2 more)

### Community 119 - "FCMService"
Cohesion: 0.21
Nodes (5): PendingMessagesResponseDTO, SMSPayload, FCMService, FirebaseMessagingService, RemoteMessage

### Community 120 - ".doWork"
Cohesion: 0.52
Nodes (4): Context, SendTiming, SMSHelper, Result

### Community 121 - ".handleSendSms"
Cohesion: 0.26
Nodes (8): FCM_SEND_SKIPPED_ERROR_CODE, normalizeId(), parseIdList(), shouldSkipFcmSend(), skippedBatchResponse(), getFcmErrorCode(), getFcmErrorMessage(), Process

### Community 122 - "scripts"
Cohesion: 0.17
Nodes (12): scripts, build, coverage, dev, lint, start, test, test:e2e (+4 more)

### Community 123 - "ExampleInstrumentedTest.java"
Cohesion: 0.27
Nodes (7): ExampleInstrumentedTest, ExampleUnitTest, androidx.test.ext.junit.runners.AndroidJUnit4, assert, instrumentationregistry, org.junit.runner.RunWith, org.junit.Test

### Community 125 - ".receiveSMS"
Cohesion: 0.38
Nodes (8): IgnoredReceivedSmsReason, isMalformedReceivedSms(), isTooLateForWebhook(), lateArrivalTimestamps(), receivedSmsIgnoreReason(), receivedWebhookMaxAgeMs(), resolveReceivedAt(), toValidDate()

### Community 127 - "breadcrumbs.ts"
Cohesion: 0.27
Nodes (8): BreadcrumbNav(), buildBreadcrumbs(), Crumb, INDEX_TABS, LABELS, LINK_TARGETS, labels(), titleCase()

### Community 128 - "billing.ts"
Cohesion: 0.27
Nodes (9): billingPriceLabel(), BillingState, deriveBillingState(), freePayload, proPayload, YearlySwitchOffer, yearlySwitchTier(), findPlanTier() (+1 more)

### Community 129 - "StickyNotificationService"
Cohesion: 0.24
Nodes (5): Intent, StickyNotificationService, IBinder, Notification, Service

### Community 131 - "jest"
Cohesion: 0.22
Nodes (9): jest, collectCoverageFrom, coverageDirectory, moduleFileExtensions, rootDir, testEnvironment, testRegex, transform (+1 more)

### Community 132 - "batch-summary.ts"
Cohesion: 0.36
Nodes (6): BatchMessageCounts, BatchSummary, countMessageStatuses(), DerivedBatchStatus, counts(), summarizeBatch()

### Community 135 - "dependencies"
Cohesion: 0.25
Nodes (8): lodash, tslib, dependencies, msgpackr-extract, @nest-modules/mailer, pnpm, overrides, packageExtensions

### Community 136 - "SubscriptionStatus"
Cohesion: 0.25
Nodes (8): SubscriptionStatus, Active, Canceled, Incomplete, IncompleteExpired, PastDue, Trialing, Unpaid

### Community 137 - "tsconfig.build.json"
Cohesion: 0.25
Nodes (7): compilerOptions, rootDir, tsBuildInfoFile, exclude, extends, include, ./tsconfig.json

### Community 138 - "translate-safe-dom.test.tsx"
Cohesion: 0.32
Nodes (3): react-dom, patchDomForTranslation(), PATCHED

### Community 139 - "device-list.test.tsx"
Cohesion: 0.33
Nodes (5): ago(), device(), devices, PointerEventPolyfill, row()

### Community 140 - "webhooks-section.test.tsx"
Cohesion: 0.29
Nodes (3): getWebhooks, useBillingPlans, useSubscription

### Community 141 - "ErrorBoundary.tsx"
Cohesion: 0.29
Nodes (3): ErrorBoundary, ErrorBoundaryProps, ErrorBoundaryState

### Community 142 - "support-hq-widget.test.tsx"
Cohesion: 0.33
Nodes (5): destroy, fireLoad(), init, scripts(), useSessionMock

### Community 143 - "nest-cli.json"
Cohesion: 0.33
Nodes (5): collection, compilerOptions, assets, watchAssets, sourceRoot

### Community 144 - "SECURITY.md"
Cohesion: 0.33
Nodes (4): Contributing, License, Reporting a Vulnerability, Security Policy

### Community 145 - "subscription-info.test.tsx"
Cohesion: 0.33
Nodes (4): freeSubscription, proSubscription, useCurrentUser, useSubscription

### Community 146 - "app-header.test.tsx"
Cohesion: 0.33
Nodes (3): pathname, session, useSession

### Community 148 - "1. Firebase Configuration"
Cohesion: 0.40
Nodes (5): 1. Firebase Configuration, 2. API Configuration, For Development:, For Production:, Setup Instructions

### Community 149 - "gradlew"
Cohesion: 0.70
Nodes (4): gradlew script, die(), save(), warn()

### Community 150 - "checkoutSource"
Cohesion: 0.60
Nodes (3): checkoutSource(), CheckoutSourceFields, match()

### Community 151 - "recipients.ts"
Cohesion: 0.80
Nodes (3): assertValidRecipients(), describeRecipient(), findInvalidRecipients()

### Community 152 - "Self-hosting"
Cohesion: 0.40
Nodes (5): Build the Android app, Run with Docker Compose, Run without Docker, Self-hosting, What you need

### Community 153 - "Testing"
Cohesion: 0.40
Nodes (4): End-to-end tests (Playwright), Testing, The rule, Unit / component tests (Vitest + React Testing Library + MSW)

### Community 156 - "sent"
Cohesion: 0.67
Nodes (4): Architecture, Features, sent(), failed()

### Community 157 - "device-selection.test.ts"
Cohesion: 0.50
Nodes (3): ONE, THREE, TWO

## Knowledge Gaps
- **883 isolated node(s):** `GatewayStatsData`, `SMSForwardResponseDTO`, `SubscriptionPlan`, `SubscriptionUsage`, `GREEN` (+878 more)
  These have ≤1 connection - possible missing edges or undocumented components. (Counts symbols only; 1453 node(s) total have ≤1 connection when file, concept and rationale nodes are included.)
- **30 thin communities (<3 nodes) omitted from report** — run `graphify query` to explore isolated nodes.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `page()` connect `mock-api.ts` to `EmailController`, `gateway.service.ts`, `email-render.ts`?**
  _High betweenness centrality (0.215) - this node is a cross-community bridge._
- **Why does `Phase 6 — Background Services & Receivers ✅ Complete` connect `SMSDTO` to `update-app-helpers.ts`, `.getSharedPreferenceString`, `What's Done`, `MainActivity.java`?**
  _High betweenness centrality (0.096) - this node is a cross-community bridge._
- **Why does `errorMessage()` connect `update-app-helpers.ts` to `device-list.tsx`, `SMSDTO`?**
  _High betweenness centrality (0.096) - this node is a cross-community bridge._
- **What connects `GatewayStatsData`, `SMSForwardResponseDTO`, `SubscriptionPlan` to the rest of the system?**
  _883 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `DashboardScreen.kt` be split into smaller, more focused modules?**
  _Cohesion score 0.04180645161290322 - nodes in this community are weakly interconnected._
- **Should `@nestjs/common` be split into smaller, more focused modules?**
  _Cohesion score 0.03432559464463752 - nodes in this community are weakly interconnected._
- **Should `cn` be split into smaller, more focused modules?**
  _Cohesion score 0.039194139194139194 - nodes in this community are weakly interconnected._