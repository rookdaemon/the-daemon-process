# PROGRESS.md — Status Tracker

## Chapters

| Chapter | Title | Spec | Status | Constraint Check | Word Count |
|---------|-------|------|--------|------------------|------------|
| 1 | fork() | spec/spec_ch01_fork.md | Implemented | — | — |
| 2 | bind() | spec/spec_ch02_bind.md | Implemented | Pass | 601 |
| 3 | accept() | spec/spec_ch03_accept.md | Implemented | Pass | 670 |
| 4 | clock_gettime() | spec/spec_ch04_clock.md | Implemented | Pass | 667 |
| 5 | EAGAIN | spec/spec_ch05_eagain.md | Implemented | Pass | 699 |
| 6 | SIGHUP | spec/spec_ch06_sighup.md | Implemented | Pass | 699 |
| 7 | SIGTERM | spec/spec_ch07_sigterm.md | Implemented | Pass | 645 |
| 8 | Respawn | spec/spec_ch08_respawn.md | Implemented | Pass | 644 |

## Planning Artifacts

| Artifact | Status |
|----------|--------|
| CONSTRAINTS.md | Complete |
| CONTRIBUTING.md | Complete |
| CLAUDE.md | Complete |
| NARRATOR_VOICE.md | Complete |
| THEME_DOSSIERS.md | Complete |
| CONCEPTUAL_MAP.md | Complete |
| EMOTIONAL_ARC.md | Complete |
| THEME_TRACKER.md | Complete |
| domain/unix_process_lifecycle.md | Draft |

## Log

- **2026-01-31**: Chapter 2 (bind()) spec created and chapter written by W-3. All CONSTRAINTS.md forbidden word checks pass. 601 words.
- **2026-01-31**: Chapter 3 (accept()) spec created and chapter written by W-4. All CONSTRAINTS.md forbidden word checks pass. 670 words.
- **2026-01-31**: Chapter 5 (EAGAIN) spec created and chapter written by W-6. All CONSTRAINTS.md forbidden word checks pass. 699 words.
- **2026-01-31**: Chapter 6 (SIGHUP) spec created and chapter written by W-7. All CONSTRAINTS.md forbidden word checks pass. 699 words.
- **2026-01-31**: Chapter 7 (SIGTERM) spec created and chapter written by W-8. All CONSTRAINTS.md forbidden word checks pass. 645 words.
- **2026-01-31**: Chapter 8 (Respawn) spec created and chapter written by W-9. Echoes Chapter 1 structure with PID 11438, stale pidfile read of "7291". All CONSTRAINTS.md forbidden word checks pass. 644 words.
- **2026-01-31**: THEME_DOSSIERS.md created by W-12. Chapter-by-chapter thematic guidance covering primary themes, key motifs, philosophical undercurrents, and inter-chapter connections for all 8 chapters.
- **2026-01-31**: EMOTIONAL_ARC.md created by W-14. Defines reader emotional state per chapter, precision-as-affect mechanism, and philosophical confrontation trajectory across 4 acts.
- **2026-01-31**: THEME_TRACKER.md created by W-15. Tracks 8 motifs across all 8 chapters with distribution table, per-motif detail tables, and thematic arc summaries.
- **2026-01-31**: Chapter 2 (bind()) updated by W-23. Added epoll_create1(0) and epoll_ctl() initialization to establish the epoll instance (fd 5) before the event loop in Chapter 3. Technical accuracy fix.
- **2026-01-31**: Chapters 1-6 harmonized by W-1 (G-22). Unified PID to 48891, port to 8402, fd layout to fd3=log/fd4=socket/fd5=epoll/fd6=timer, epoll_wait throughout. Removed duplicate socket setup from Ch1, removed unexplained database socket from Ch3, replaced poll() with epoll_wait in Ch4. All CONSTRAINTS.md checks pass.
- **2026-01-31**: Chapters 3, 5, 6, 8 updated by W-2. Fixed monotonic timestamp inconsistency: converted raw nanosecond format to tv_sec/tv_nsec struct format, aligned all values to coherent ~14.8M second timeline (~171 days system uptime). Ch8 log duration reference corrected from 94877s to 29650s.
- **2026-01-31**: Ch1 spec updated by W-4 (G-31). Removed pidfile write requirement from Ch1 spec (AC-8, AC-1, Technical Requirements, Cross-cutting Concerns) since G-22 removed the pidfile write from Ch1 prose and Ch2 already handles the pidfile motif with the richer stale-PID discovery theme.
- **2026-01-31**: Chapters 7-8 harmonized by W-3 (G-30). Ch7: replaced poll() with epoll_wait(), fixed timer fd 5→6, added epoll fd 5 close, PID 7291→48891, port 8080→8402. Ch8: stale pidfile PID 7291→48891 with corrected ASCII bytes, port 8080→8402, replaced poll() with epoll_wait(), added epoll_create1/timerfd_create for fd5/fd6. Specs updated. All CONSTRAINTS.md checks pass.
- **2026-01-31**: Chapter 2 (bind()) updated by W-5 (G-32). Fixed CLOCK_MONOTONIC timestamp from 14227s (~4h uptime) to 14823091s (~171d uptime), aligning with Ch1's established timeline.
