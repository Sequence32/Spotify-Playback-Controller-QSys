# Spotify Playback Controller for Q-SYS

A professional Q-SYS plugin that provides full Spotify playback control through the Spotify Web API. Features OAuth PKCE authentication (no client secret needed), transport controls, now-playing metadata with album art, playlist browsing, and volume control.

![Q-SYS](https://img.shields.io/badge/Q--SYS-Plugin-blue)
![Spotify](https://img.shields.io/badge/Spotify-Web%20API-1DB954)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Features

- **OAuth PKCE Authentication** — Secure login without exposing client secrets
- **Built-in Login Page** — Browse to the Core's IP to get a styled Spotify login button
- **Transport Controls** — Play, Pause, Next, Previous, Shuffle, Repeat
- **Now Playing** — Track name, artist, album, progress bar, and duration
- **Album Art** — URL output compatible with Q-SYS UCI Media Display
- **Playlist Browsing** — View and play your top 10 playlists with cover art
- **Volume Control** — Bidirectional volume with feedback
- **Seek** — Seek to any position in the current track
- **Auto Token Refresh** — Tokens refresh automatically before expiry

---

## Prerequisites

- **Q-SYS Designer** 9.x or later
- **Spotify Premium** account (required for Web API playback control)
- A registered app in the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)

---

## Setup Guide

### Step 1 — Create a Spotify Developer App

1. Go to the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard)
2. Log in with your Spotify account
3. Click **Create App**
4. Fill in:
   - **App Name**: anything (e.g., `Q-SYS Controller`)
   - **App Description**: anything
   - **Redirect URI**: `http://127.0.0.1:9091/callback`
   - **APIs Used**: check **Web API**
5. Click **Save**
6. Copy the **Client ID** from the app overview page

> **Important:** The redirect URI must match **exactly**. For emulation/development, use `http://127.0.0.1:9091/callback`. For a deployed Core on the LAN, use `http://<core-ip>:9091/callback` — but note that Spotify requires HTTPS for non-loopback addresses, so `127.0.0.1` is recommended for most use cases.

### Step 2 — Install the Plugin in Q-SYS Designer

1. Copy `Spotify_Playback_Controller.qplug` to your Q-SYS plugins directory
2. Open your design in Q-SYS Designer
3. Find **Spotify Playback Controller** in the plugin list and drag it into your design

### Step 3 — Configure Plugin Properties

In the plugin's **Properties** panel, set:

| Property | Value | Notes |
|----------|-------|-------|
| **Client ID** | `your-spotify-client-id` | From the Spotify Developer Dashboard |
| **Core IP Address** | `127.0.0.1` | Use `127.0.0.1` for emulation, or the Core's IP when deployed |
| **Redirect Port** | `9091` | Must match the port in your Spotify redirect URI |
| **Poll Interval (s)** | `3` | How often to poll for now-playing updates (1–10 seconds) |

### Step 4 — Authorize

1. Save/Emulate your design so the plugin is running
2. Press the **Authorize** button on the plugin block
3. The **Auth URL** field will show an address like `http://127.0.0.1:9091`
4. Open that address in any web browser on the same machine
5. Click the green **"Log in with Spotify"** button
6. Log in to Spotify and approve the permissions
7. You'll see a "Spotify Authorized!" confirmation page
8. Back in Q-SYS, the **Auth Status** LED turns green and your **User Name** appears

---

## Controls Reference

### Authentication
| Control | Type | Description |
|---------|------|-------------|
| `Authorize` | Trigger | Start the OAuth login flow |
| `Logout` | Trigger | Clear tokens and stop polling |
| `Auth_Status` | LED Indicator | Green = authenticated, Red = logged out, Yellow = waiting |
| `Auth_URL` | Text (output) | Login page URL to open in a browser |
| `Auth_Code_Input` | Text (input) | Fallback: manually paste an auth code |
| `User_Name` | Text (output) | Logged-in Spotify display name |

### Transport
| Control | Type | Description |
|---------|------|-------------|
| `Play` | Trigger | Resume playback |
| `Pause` | Trigger | Pause playback |
| `Next` | Trigger | Skip to next track |
| `Previous` | Trigger | Go to previous track |
| `Shuffle` | Toggle | Enable/disable shuffle mode |
| `Repeat` | Trigger | Cycle repeat: off → context → track → off |
| `Repeat_State` | Text (output) | Current repeat mode |
| `Is_Playing` | Indicator | True when playback is active |

### Now Playing
| Control | Type | Description |
|---------|------|-------------|
| `Track_Name` | Text (output) | Current track name |
| `Artist_Name` | Text (output) | Artist(s), comma-separated |
| `Album_Name` | Text (output) | Album name |
| `Album_Art_URL` | Text (output) | HTTPS URL for album artwork (use with UCI Media Display) |
| `Progress_Ms` | Knob 0–100% (output) | Playback position as percentage |
| `Progress_Text` | Text (output) | Formatted as `2:34 / 4:12` |
| `Duration_Ms` | Text (output) | Track duration in milliseconds |

### Volume & Device
| Control | Type | Description |
|---------|------|-------------|
| `Volume` | Knob 0–100 (bidirectional) | Get/set Spotify volume |
| `Active_Device` | Text (output) | Name of the active Spotify Connect device |
| `Device_ID` | Text (output) | Device ID (for debugging) |
| `Seek_Position` | Knob 0–100% (input) | Seek to a percentage of the track |

### Playlists
| Control | Type | Description |
|---------|------|-------------|
| `Refresh_Playlists` | Trigger | Fetch/refresh the user's playlists |
| `Playlist_Name 1–10` | Text (output) | Playlist names |
| `Playlist_Art 1–10` | Text (output) | Playlist cover art URLs (use with UCI Media Display) |
| `Playlist_Play 1–10` | Trigger | Start playing that playlist |

---

## UCI Integration

### Album Art on Touch Panels

To display album artwork on a Q-SYS UCI:

1. Add a **Media Display** element to your UCI layout
2. Set its source to the `Album_Art_URL` control (or any `Playlist_Art` control)
3. The plugin outputs HTTPS image URLs that Q-SYS can render natively

### Playlist Cover Art

Each `Playlist_Art 1–10` control contains the cover image URL for the corresponding playlist. Wire these to individual Media Display elements to show playlist thumbnails alongside the playlist names.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **"redirect_uri: Not matching"** | Ensure the redirect URI in Spotify Dashboard matches exactly: `http://127.0.0.1:9091/callback` |
| **"redirect_uri: Insecure"** | Spotify requires HTTPS for non-loopback IPs. Use `127.0.0.1` instead of a LAN IP |
| **"Invalid authorization code"** | Auth codes are single-use. If you see this once after login, it's a harmless duplicate request — check if the plugin actually authenticated (username showing) |
| **Transport buttons not working** | Ensure you have an active Spotify session on at least one device (phone, desktop app, etc.) |
| **No album art showing** | Verify the UCI element is set to "Media Display" type and is wired to the `Album_Art_URL` control |
| **Plugin crashes on Authorize** | Check that the Client ID is set in Properties and the Core IP Address is correct |
| **Token expires after 1 hour** | This is handled automatically — the plugin schedules a refresh 60 seconds before expiry |

---

## Architecture

```
┌─────────────────────────────────────────────────────┐
│  Q-SYS Core / Designer Emulation                    │
│                                                     │
│  ┌─────────────────────────────────────────────┐    │
│  │  Spotify Playback Controller Plugin         │    │
│  │                                             │    │
│  │  TcpSocketServer (:9091)                    │    │
│  │    ├── Serves login landing page            │    │
│  │    └── Catches OAuth redirect callback      │    │
│  │                                             │    │
│  │  HttpClient                                 │    │
│  │    ├── Token exchange & refresh             │    │
│  │    ├── Playback control (PUT/POST)          │    │
│  │    └── Metadata polling (GET)               │    │
│  └─────────────────────────────────────────────┘    │
│                                                     │
│  Browser (any device)                               │
│    └── Opens login page → Spotify OAuth → Redirect  │
└─────────────────────────────────────────────────────┘
         │                          │
         ▼                          ▼
  Spotify Auth Server        Spotify Web API
  (accounts.spotify.com)     (api.spotify.com)
```

---

## Security Notes

- **No client secret stored** — Uses PKCE (Proof Key for Code Exchange), which is the recommended flow for public/embedded clients
- **Tokens are memory-only** — Access and refresh tokens are not persisted to disk; a re-auth is needed after a Core reboot
- **Scopes are minimal** — Only requests permissions needed for playback control and playlist reading

---

## License

MIT License — See [LICENSE](LICENSE) for details.

---

## Author

**Dustin Bennett** — [TEL Systems](https://telsystems.com)
