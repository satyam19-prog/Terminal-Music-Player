# 🎵 Terminal Music Player (macOS)

A robust, terminal-based audio player built entirely in Node.js for macOS. It demonstrates core operating system and software engineering principles including **CLI development**, **file system I/O**, and **low-level process management** using the native macOS `afplay` binary and POSIX process signals.

---

## 📋 Project Overview

- **Runtime**: Node.js (v18+)
- **Operating System**: macOS
- **Audio Engine**: Native macOS `/usr/bin/afplay` executed via `child_process.spawn` (Zero npm audio packages required).
- **UI & UX**: Interactive terminal interface with colored status indicators (`chalk`) and interactive arrow-key selection (`enquirer`).

---

## 📂 Project Structure

```text
Terminal_Application/
├── songs/                  # Local directory containing audio files (.mp3, .wav)
│   ├── Funk.wav
│   ├── Glass.wav
│   └── Submarine.wav
├── index.js                # Core application (File handling, CLI, & Process logic)
├── package.json            # Node.js manifest and script definitions
└── README.md               # Documentation & professor presentation guide
```

---

## 🎯 Grading Criteria Alignment

### 1. CLI Development
- **Dashboard UI**: Renders a clean terminal layout with an ASCII banner, playback state card, and track directory.
- **Contextual State Badges**:
  - `▶ PLAYING` (Bright Green)
  - `⏸ PAUSED` (Yellow)
  - `⏹ STOPPED` (Gray)
- **Interactive Action Menu**: Arrow-key navigability for:
  - Play / Resume
  - Pause Playback
  - Stop Playback
  - Next Track (`⏭`)
  - Previous Track (`⏮`)
  - Select Specific Song (`📋`)
  - Rescan Directory (`🔄`)
  - Exit Player (`🚪`)
- **Active Track Indicators**: Marks the currently active track with `▶` in the playlist table.

### 2. File Handling (`node:fs` and `node:path`)
- **Directory Validation & Auto-Creation**: Checks if `./songs` exists via `fs.existsSync`. If absent, automatically generates it using `fs.mkdirSync`.
- **Extension Filtering**: Reads directory contents via `fs.readdirSync` and filters only valid audio files (`.mp3` and `.wav`) using case-insensitive validation.
- **Metadata Inspection**: Uses `fs.statSync` to compute and display formatted file sizes (`KB` / `MB`).
- **Defensive Error Handling**:
  - Handles empty folders with guidance on how to populate songs.
  - Verifies file presence on disk before playback to avoid crashes if files are moved/deleted.

### 3. Process Management (`node:child_process`)
- **Native Process Spawning**:
  ```javascript
  this.audioProcess = spawn('afplay', [song.path], { stdio: 'ignore' });
  ```
- **POSIX Signal Control**:
  - **Pause (`SIGSTOP`)**: Suspends CPU execution of `afplay` at the kernel level, instantly freezing audio without killing the process:
    ```javascript
    this.audioProcess.kill('SIGSTOP');
    ```
  - **Resume (`SIGCONT`)**: Instructs the OS scheduler to resume execution of `afplay`, continuing playback from the exact millisecond it was frozen:
    ```javascript
    this.audioProcess.kill('SIGCONT');
    ```
  - **Stop (`SIGTERM`)**: Gracefully terminates the running `afplay` instance before starting new songs or exiting.
- **Auto-Play Progression**: Listens to the child process `close` event (`code === 0`). When a song reaches its natural end, the player advances to the next track automatically.
- **Zombie / Orphan Prevention**: Registers hooks for `SIGINT` (Ctrl+C), `SIGTERM`, and `process.exit` to guarantee no orphan `afplay` background processes linger in macOS Activity Monitor.

---

## 🚀 Quick Start Guide

### 1. Install Dependencies
```bash
npm install
```

### 2. Add Audio Files
Place any `.mp3` or `.wav` files into the `./songs` directory (sample `.wav` files are pre-loaded).

### 3. Run the Application
```bash
npm start
```
*or directly with:*
```bash
node index.js
```

---

## 🎓 Talking Points for Professors

1. **Why `child_process.spawn` instead of `exec`?**
   - `spawn` is stream-based and asynchronous. It immediately provides a handle to the `ChildProcess` object and its PID, allowing real-time signal control (`SIGSTOP`, `SIGCONT`, `SIGTERM`), whereas `exec` buffers all output in memory until process termination.

2. **How does Pausing work without an audio library?**
   - Instead of decoding audio frames or manipulating buffers in JavaScript, the application leverages Unix POSIX signals. Sending `SIGSTOP` causes the macOS kernel to halt the `afplay` process execution thread. Sending `SIGCONT` resumes it without state loss.

3. **How are race conditions handled between manual stopping and auto-advance?**
   - The application maintains an `isIntentionalStop` flag. When the user skips or stops, `isIntentionalStop` is flagged `true` before issuing the kill signal. This prevents the `close` event listener from falsely recognizing the stop as a naturally finished song.
