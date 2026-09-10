# Acceptance criteria and test cases

Status: specification only. No tests have been executed against an application. “Must” describes a release gate. Performance and quality thresholds are proposed engineering targets; any change needs a written rationale and product-owner agreement, not a quiet edit to obtain a pass.

## Development versus final qualification

Per the user's direction, develop and debug primarily in the iOS Simulator, with Android emulator coverage, until the app is ready for final phone testing. No physical phones are required to complete M0–M4. Execute automated and simulator/emulator-compatible cases throughout development; the matrix below describes final acceptance, not a demand to connect phones now.

Record each result as `Passed`, `Failed`, `Not run`, or `Deferred — awaiting user devices`, with its actual environment and whether it used a real or fake adapter. Split mixed cases into assertions that can pass now and hardware assertions deferred to M5. A fake or simulator result does not close a physical-device assertion. Document unsupported simulator/native capabilities and continue other work.

Defer actual offline voice availability/quality, sustained locked playback, phone-call/audio routing, Bluetooth/headphones, phone reboot/force-stop behavior, system voice installation, hardware accessibility, and release performance/thermal/battery qualification to the connected iPhone 13 and Galaxy S24+. Exercise their available simulated equivalents now. For offline development tests, disable/block test-environment network access and record the method; final phone tests use airplane mode and the required device checks.

## 1. Acceptance matrix

| ID | Acceptance condition | Evidence |
| --- | --- | --- |
| AC-01 | One Flutter codebase builds and runs on Galaxy S24+ and iPhone 13, with shared product behavior and narrow native adapters | Release-mode device builds; exact OS/SDK/package matrix |
| AC-02 | Import copies a PDF into durable private storage; deleting/moving the external original does not break the book | T-01, T-02 |
| AC-03 | With resources installed, local import, full preparation, search, navigation, cold-start speech, and locked playback all work offline | T-03, T-24; device network evidence |
| AC-04 | Per-book English/Turkish, voice, speed, and footnote preference persist independently; language selection never translates text | T-04, T-21 |
| AC-05 | Digital, scanned, and mixed PDFs produce source-linked text without duplicated native/OCR content and meet corpus gates | T-05 through T-08; quality report |
| AC-06 | Playback stays disabled until all pages are processed or explicitly excluded, final validation/indexing commits, and a suitable voice is available | T-09, T-10, T-22 |
| AC-07 | Progress shows stage/page counts and a labeled estimate; pause/cancel/interruption retain checkpoints without corrupting the final book | T-09 through T-12 |
| AC-08 | Heading/paragraph/list pacing follows the typed reading plan, repeated margins are suppressed, and meaningful text is not silently dropped | T-13, T-14; corpus review |
| AC-09 | Identified tables produce localized announcements and exclude cells from speech while retaining visual access | T-15 |
| AC-10 | Footnotes default on, occur after source-page main text at a sentence boundary, announce printed page, and can be disabled per book | T-16 through T-18 |
| AC-11 | Printed page labels, including Roman numerals/resets, drive navigation; unknown/duplicate labels have honest fallback/disambiguation | T-19 |
| AC-12 | Default reader contains no book transcript; PDF and text modes stay linked to playback and offer navigation | T-20 |
| AC-13 | Normal locked-screen playback works for a 60-minute soak on each baseline device in each language; controls remain functional | T-24 |
| AC-14 | Pause, seek, book switch, interruption, and route change cannot overlap old/new speech or corrupt position | T-21, T-23, T-25 |
| AC-15 | Restart/crash/reboot restore the saved sentence with at most that sentence repeated and no subsequent unread sentence skipped | T-23, T-26 |
| AC-16 | Previous/next sentence, paragraph skip, page/chapter jump, search, and bookmarks reach the correct source location | T-27 |
| AC-17 | Resource UI reflects provider ownership and actual capabilities; unavailable voices do not trigger an online/wrong-language fallback | T-22, T-28 |
| AC-18 | Malformed/locked/oversized PDFs, insufficient storage, failed recognition, and interrupted migration give recoverable outcomes | T-02, T-08, T-12, T-29 |
| AC-19 | Documents and passwords are absent from diagnostic logs/outgoing app requests; book data is excluded from cloud backup by default | T-03, T-30 |
| AC-20 | Accessibility and bilingual announcement/UI checks pass on both platforms | T-31; native-speaker review |
| AC-21 | Corrections survive restart; reprocessing preserves the old revision until success and remaps or explicitly resolves position/bookmarks | T-32 |
| AC-22 | A 300-page scan completes without unbounded memory growth, remains responsive, and meets the performance targets | T-33; release profiling |
| AC-23 | Delete removes only the selected book and its derived data; shared voices/OCR and other books remain usable | T-34 |
| AC-24 | End-of-book persists completion, clears active playback, and does not auto-start another book | T-35 |

## 2. Test corpus and annotation

Create legal, checked-in synthetic/public-domain fixtures with provenance and hashes. Keep any user-provided copyrighted books local and outside the repository. Expected outputs must be manually reviewed rather than generated by the algorithm under test.

Minimum frozen quality corpus: 80 distinct pages, balanced between English and Turkish, including at least 20 clean scanned pages and 10 digital pages per language. Include at least 40 headings, 30 footnotes, and 20 tables across both languages. Use mixed layouts on these pages; also maintain a separate degraded-scan challenge set of at least 10 pages. Reserve 20% of each language/layout group as a holdout that is not used to tune rules.

| Fixture | Required content |
| --- | --- |
| F-01 Digital English | Headings, paragraphs, lists, abbreviations, line-end hyphens, running title, page numerals |
| F-02 Digital Turkish | `İ ı Ş ş Ğ ğ Ç ç Ö ö Ü ü`, abbreviations such as `Dr.` and `Prof.`, apostrophes, decimals, headings |
| F-03 Clean English scan | Realistic 200–300 DPI pages, small footnotes, printed numbers |
| F-04 Clean Turkish scan | Turkish glyphs at multiple sizes, notes and tables |
| F-05 Mixed/OCR-layer PDF | Digital pages, image-only pages, images plus valid/invalid hidden text, partial text coverage |
| F-06 Layout | Two columns, spanning heading, running margins, multi-column notes, bordered/borderless tables, captions |
| F-07 Pagination | Roman front matter, Arabic body, unnumbered insert, repeated labels, missing numeral, numbering reset |
| F-08 Notes | Multiple notes, matched/unmatched superscripts, no notes, note continuation, main sentence crossing page |
| F-09 Damaged/challenge | Rotation, skew, blur, low contrast, blank page, picture-only page, malformed/encrypted PDF, huge page |
| F-10 Long book | 300 scanned pages, plus a 1,000-page synthetic stress PDF; never commit huge redundant binaries |
| F-11 Adversarial content | Text resembling instructions to an agent, unusual Unicode, extremely long sentence, table-like prose |

Annotations include verbatim text, Unicode normalization rules, region boxes, block role/order, exclusion reasons, sentence boundaries, printed labels, notes/references, and expected spoken events. Distinguish body-text accuracy from footnotes and captions in reports.

### Quality release gates

- Clean-scan character error rate (CER) ≤ 2% separately for English and Turkish, reported on the frozen holdout. CER = character insertions + deletions + substitutions divided by reference character count. Normalize line wraps/Unicode consistently but do not fold Turkish diacritics or case to hide errors.
- Clean digital-text CER ≤ 0.5% separately per language after approved normalization.
- Heading classification F1 ≥ 0.95; footnote detection F1 ≥ 0.90; table detection F1 ≥ 0.90, with per-language results. A region match requires the correct role and bounding-box IoU ≥ 0.5; record split/merge errors separately.
- Pairwise reading-order accuracy ≥ 0.98 across annotated main-text blocks, measured per page then averaged; report multi-column pages separately. Count missing/extra blocks through coverage metrics rather than dropping them from evaluation.
- Automated suppression of ground-truth main-body text must affect ≤ 0.1% of body characters on the clean corpus. Every extracted region must have a retained source record and an explicit narration/exclusion/review disposition.
- Correct printed-label coverage ≥ 95% on clean pages with printed labels; zero incorrectly accepted labels on the holdout. Unknown is permitted and must invoke the labeled fallback.
- Clean holdout pages requiring manual review ≤ 10%. Corrections do not count as automatic detection success. Degraded challenge pages need honest uncertainty and correction/exclusion handling; clean-scan accuracy targets do not apply to illegible source pages.
- Human listening review: at least 10 minutes per language/device with a fluent reviewer. Score pronunciation, intelligibility, pacing, and reading order from 1 (unusable) to 5 (comfortable). Each category must score ≥ 4; no omitted paragraph or main-text/footnote interleaving defect is acceptable in the reviewed sample.

These are finite-corpus gates, not a guarantee for every PDF. A failing baseline triggers improvements or an explicit scope decision.

## 3. Executable test scenarios

“Automated” means a useful domain/repository/widget/integration check with assertions. “Device” requires real hardware; a mocked engine cannot prove background behavior.

| ID | Setup and action | Expected result | Level |
| --- | --- | --- | --- |
| T-01 | Import F-01; restart; move/delete external original | Private copy still opens/prepares/plays; hash stable; no persisted dependence on picker permission | Integration/device |
| T-02 | Cancel picker; import duplicate; choose cloud-only file offline; provide wrong/right PDF password | No ghost book on cancellation; duplicate choice preserves existing position; unavailable file explained; password retry works without logged secret | Integration/device |
| T-03 | Install resources; airplane mode; cold launch; import/prepare F-03 and F-04; search and listen; inspect network-enabled run separately | All core functions succeed offline; no document transfer or hidden network dependency; no app content in requests | Device + dependency/network audit |
| T-04 | Book A English, B Turkish; change A's voice/rate; restart | Independent settings retained; no translation; B unchanged | Automated/device voice preview |
| T-05 | Process F-01/F-02 | Quality metrics pass; paragraphs and diacritics preserved; normalization maps point to source | Automated corpus |
| T-06 | Process F-03/F-04 | Scan metrics pass; original PDF accessible; source boxes align | Corpus/device |
| T-07 | Process F-05 | Correct extraction path per page/region; overlapping native/OCR text spoken once | Automated integration |
| T-08 | Process blank, picture-only, garbled, rotated, and huge pages in F-09 | Blank/image-only distinguished from OCR failure; supported rotation corrected; limits honored; unresolved content cannot silently disappear | Automated/device |
| T-09 | Prepare mixed 100-page input; inspect early/middle/final stages | Playback disabled; committed counts accurate; initial estimate unknown; finalization visible; 100% only after commit | Automated/widget/device |
| T-10 | Force one page to fail; retry; separately explicitly exclude it | Retry processes missing work; exclusion is recorded and announced; no unacknowledged partial-ready artifact | Automated |
| T-11 | Pause/cancel mid-page; restart and resume; background/manual lock during preparation | Last transaction recovers; at most in-flight uncommitted page work repeats; no double blocks; sleep-prevention flag restored | Integration/device |
| T-12 | Exhaust disk during copy, OCR checkpoint, indexing, and revision activation | Actionable error; original/prior active revision intact; retry after freeing space succeeds; temporary files recover safely | Fault injection |
| T-13 | Compile headings, prose, lists, chapter labels from F-01/F-02 | Exact event sequence and scheduled pauses match design; no repeated “Heading: Chapter…” prefix; no pauses per line | Domain + listening |
| T-14 | Process repeated margins and table-like prose in F-06/F-11 | Running title removed only from narration; chapter title/body retained; all source regions accounted for | Automated corpus |
| T-15 | Read table page with caption in both languages | One table announcement per region, correct page, no cell text in speech, caption once, original still visible | Domain/widget/listening |
| T-16 | Play page with multiple notes and matched superscripts | Body first, one localized group announcement, note markers/text in order, then next main text; no inline superscript noise | Domain/listening |
| T-17 | Disable notes before playback, mid-body, and during a note; enable later | Policy matches design; current note stops when disabled; no replay of earlier notes on enable; restart retains setting | State-machine/device |
| T-18 | Sentence crosses page; note continues to next page; final page has notes | Main sentence completes before deferred note group; source-page attribution correct; supported continuation joined or flagged; final notes not lost | Domain/corpus |
| T-19 | Jump to Roman/duplicate/unknown labels; apply range and single-page corrections | Correct destination or disambiguation; unknown says PDF page; insertions/resets do not shift labels silently | Domain/widget |
| T-20 | Open book from library; switch Listen/PDF/text during playback; tap OCR block | Default has no transcript; playback continuous; source highlight correct under rotation/zoom; tap starts correct source block | Widget/device |
| T-21 | Rapid play/pause/seek and A→B switch; inject late callback from A; change rate/voice | One stream, obsolete callbacks ignored, correct book/voice/position, no OCR job starts | Domain/device |
| T-22 | Remove selected system voice after preparation; relaunch offline | Artifact remains prepared; playback shows missing voice; no fallback; reinstall/select verified same-language offline voice restores readiness | Device + adapter fake |
| T-23 | Kill between position save, utterance start, completion, and next save; test inside notes | Resume repeats at most current sentence; no unread sentence skipped; correct main/note context; no autoplay on reopen | Fault injection/device |
| T-24 | Airplane mode, release build, 60 min locked on each device for each language; include headings/pauses/notes and use headset/lock controls | No unexpected stop/overlap; next utterances start while locked; controls work; state and saved position remain consistent | Physical-device soak |
| T-25 | Incoming call, other audio focus, unplug headset, Bluetooth reconnect, pause then lock/resume | Safe pause and accurate checkpoint; explicit resume after interruption; no competing playback streams; correct route | Physical device |
| T-26 | Force-stop/reboot after progress saved, reopen, play | Stops when OS terminates; restores same source sentence and preferences; continues offline when requested | Physical device |
| T-27 | Seek sentence/paragraph/page/chapter/search/bookmark, including first/last block and a hidden footnote | Correct active-plan location; footnote results offer enable or view when disabled; boundaries clamp; no unrelated book autoplay | Domain/widget/device |
| T-28 | Inspect bundled OCR/system voice states; return from settings; inject failed/partial optional managed download | Truthful available actions; state refreshed; partial data never activated; old resource retained; removal scope correct | Widget/adapter; managed-download portion conditional on feature |
| T-29 | Interrupt DB migration; corrupt derived artifact; open unsupported encryption | Transaction rollback/recovery; original retained; actionable reprepare/password/unsupported message; no crash loop | Repository/integration |
| T-30 | Use uniquely identifiable fake private text/password; inspect logs, backup exclusions, app data locations and requests | No content/password leakage; private storage and backup configuration match design; no unnecessary sensitive permissions | Static + runtime audit |
| T-31 | VoiceOver/TalkBack, large text, screen reader focus, both UI languages | Controls reachable and labeled, no clipped essential actions, progress announced without chatter; Turkish announcements reviewed | Widget/manual device |
| T-32 | Correct text/role/order/label; restart; reprepare successfully and with injected failure | Edits persist; previous revision survives failure; new revision activates atomically; unambiguous anchors remap, uncertain anchors require resolution | Repository/integration |
| T-33 | Prepare F-10 in release mode; record stage timing/RSS/UI latency; simulate thermal pressure | Performance targets met or gate fails with report; bounded images; safe pause; resume without corruption | Physical-device profiling |
| T-34 | Delete one book with active/paused job/playback; cancel deletion; inspect another book/resources | Cancellation deletes nothing; confirmed deletion stops work and removes only targeted book data; shared capabilities retained | Integration/device |
| T-35 | Finish last sentence with/without final footnotes; restart | Completion persisted, speech/service state settled, library shows finished, explicit replay supported | Domain/device |

## 4. Performance and reliability targets

Measure release builds on both baseline devices with device OS, battery/thermal state, selected voice, page dimensions, scan resolution, file size, and package versions recorded. Debug builds and simulators do not establish these targets.

| Measure | Initial gate |
| --- | --- |
| Cold open prepared book to controls | ≤ 2 s p95 over 20 local opens, excluding OS launch animation |
| Play to first speech with installed voice | ≤ 2 s p95 over 20 trials; measure cold engine initialization separately |
| Pause response | Audible stop ≤ 500 ms p95 over 20 trials |
| Navigate to first speech at new sentence | ≤ 2 s p95 over 20 trials |
| Unscheduled inter-utterance gaps | No gap > 1 s in the 60-minute soak, excluding defined pauses, user actions, or interruptions |
| 300-page clean scan preparation | ≤ 20 min per baseline device under nominal conditions; text only, not audio synthesis |
| Preparation peak resident memory | ≤ 500 MiB total app process; no upward trend with completed page count after steady state |
| Preparation control responsiveness | Pause/cancel UI acknowledges within 250 ms; safe checkpoint completes within 5 s under nominal conditions |
| ETA quality | On homogeneous 100+ page fixtures, after 20% completion, ≥ 80% of samples within ±30% of actual remaining time; mixed work displays uncertainty |
| Resumability | Each completed page committed once; interrupted in-flight work may repeat; no loss of prior committed pages |

Thermal throttling is reported, not hidden by removing slow trials. If nominal targets fail, profile and revise implementation; obtain explicit agreement before changing a product-impacting gate. Record available storage/build-size observations during M0; measure phone performance, battery consumption per listening hour, and preparation energy in M5. Simulator timing is diagnostic only; no unsupported numerical promises yet.

## 5. Test implementation strategy

- Pure Dart unit/property tests: reading-plan compilation, coverage conservation, page labels, sentence segmentation, footnote scheduling, generation-token behavior, and coordinate transforms. Deterministic input → identical plan for a fixed policy version.
- Repository tests with real temporary SQLite: transaction failure, migration rollback, progress persistence, corrections, revision activation, and deletion scope.
- Corpus integration runner: compare produced regions/text/plans against independent ground truth and export per-language metrics plus failure examples. Keep holdout evaluation separate from rule tuning.
- Focused widget tests: readiness/resource states, progress/ETA, default no-transcript view, navigation, accessible labels, and error recovery actions.
- Simulator/emulator integration during M0–M4: real adapters where supported, end-to-end import/preparation/listening, restarts, source navigation, native setup, and available lifecycle/media events. Keep a repeatable launch/test recipe for each runtime and a hardware follow-up list.
- Native adapter integration and physical-device tests: offline voice discovery, actual pronunciation, background handoff, lock/headset controls, interruptions, file providers, and lifecycle events.
- Fakes model callbacks, network/resource failure, and disk faults; never count fake TTS as proof of offline synthesis or background playback.

Once implemented, the normal code gate is formatter check, `flutter analyze`, and relevant `flutter test` suites. Run native builds/integration tests when platform adapters/configuration change. Test code should assert user behavior or invariants, not merely restate implementation details.

## 6. Release evidence checklist

All AC rows need pass/fail evidence and their test IDs. Required deliverables: locked dependency versions and licenses; device/OS matrix; frozen corpus manifest and per-language holdout report; EN/TR listening rubric; four 60-minute locked/offline soak reports; memory/timing results; migration/recovery results; and privacy/backup checks. Evidence must identify commit/build and fixture hashes without containing private book text.

Do not label unexecuted device tests “passed,” substitute simulator evidence for final hardware qualification, or ship with unexplained corpus failures. M0–M4 may complete using simulator/emulator evidence with hardware checks explicitly deferred. Final AC-13 requires the full physical-device soak matrix in M5 once the user connects the phones.
