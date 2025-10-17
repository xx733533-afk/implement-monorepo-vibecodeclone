# VibrCode Clone Application Flow Document

## Onboarding and Sign-In/Sign-Up

When a brand-new user arrives at the VibrCode Clone platform, they first encounter a clean, informational landing page that briefly explains the key capabilities of the developer environment. Prominent buttons invite the user to either sign in if they already have an account or sign up for a new one. If the user chooses to create a new account, they are brought to a sign-up page where they enter their name, email address, and a secure password. Upon submitting the form, the app validates the input and communicates with the Better Auth API to store credentials in the PostgreSQL database. A confirmation message appears once registration succeeds, and the user is automatically redirected to the main dashboard.

If a returning user selects the sign-in option, they see a straightforward login form requesting email and password. Validation messages appear inline for invalid credentials. A link labeled "Forgot Password" takes the user to a recovery page where they input their email to receive a reset link. Clicking the link in the email brings them to a reset-password page where they enter and confirm a new password. Upon setting a new password, the user can sign in immediately.

Sign-out is available at any time by clicking the user avatar in the header and selecting the logout option. This clears the session and returns the user to the landing page.

## Main Dashboard or Home Page

After signing in, the user lands on the main dashboard, which features a fixed sidebar on the left, a top header bar, and a central workspace area. The sidebar lists navigation items such as Home, Editor, Terminal, and Settings. The header displays the user’s avatar, a theme toggle switch for light and dark modes, and a quick-logout icon. In the central area, a welcome message greets the user by name and provides high-level status information, such as the number of active CLI sessions or recent project activity.

At the top of the central area, tabs allow the user to jump directly into the code editor or terminal view. A clickable card in the workspace invites exploration of advanced features like CLI execution or AI-driven code suggestions. Throughout the dashboard, every interactive element gently highlights on hover, ensuring clear navigation cues.

## Detailed Feature Flows and Page Transitions

### Code Editor Flow

When the user clicks the Editor tab or selects the Editor item in the sidebar, the central workspace transforms into a full-fledged coding environment. On the left side of the workspace, a file tree shows project folders and files. Clicking any file name loads its content into the code editor pane. The editor supports syntax highlighting and auto-completion. If the user makes changes, a save button in the header of the editor pane becomes active, and clicking it sends an update request to the backend, persisting edits in the project’s data store. Users can also create new files or folders by clicking the plus icon next to the file tree heading, which brings up a small form for the new file name. Upon creation, the new file appears in the tree and opens automatically in the editor.

Navigating away from the editor to the terminal, the app preserves the open files and unsaved changes, prompting the user before discarding if they attempt to switch views without saving.

### Terminal and CLI Execution Flow

Selecting the Terminal tab reveals a responsive terminal component that fills the central workspace. At the bottom of this view, an input prompt invites the user to type any supported CLI command. When the user presses Enter, the app sends a POST request to the `/cli/run` endpoint on the server. Immediately, a WebSocket connection is opened to a `/cli/logs/:sessionId` endpoint. As the server spawns a Docker container and runs the command, stdout and stderr messages stream back through the WebSocket and appear line by line in the terminal pane.

A stop button above the terminal allows the user to terminate the running process by sending a POST request to `/cli/stop/:sessionId`. Upon stopping, the WebSocket closes gracefully and the terminal prompt becomes active again. If the user tries to execute a command that the system does not recognize or if the container fails to start, an error message is printed in the terminal pane, and the user can adjust their input or retry.

## Settings and Account Management

By clicking Settings in the sidebar or selecting the avatar menu’s Settings option, the user enters the account management area. This page is divided into three sections. The Profile section provides form fields for name and email, each with an update button that sends changes to the server. A separate Change Password subsection asks for the current password, the new password, and confirmation, enabling secure password updates.

Below the profile controls, the Appearance section offers the same light and dark mode toggle as the header. Changing the theme here immediately updates the entire interface and stores the preference for future sessions. The notification preferences area lets users enable or disable alerts for events such as completed CLI runs or AI generation results. Each preference toggle sends a quick update to the server and displays a success message once saved.

After applying any setting changes, a breadcrumb link at the top allows the user to return to the Home dashboard or switch directly to the Editor or Terminal.

## Error States and Alternate Paths

Invalid credentials during login or sign-up trigger inline error messages explaining the issue, such as "Password must be at least eight characters" or "Email not found." If a user’s session expires while on any page, the next API call detects the invalid token and redirects them automatically to the sign-in page with a notification that the session has timed out.

When a CLI command fails due to permission issues or container errors, the terminal pane displays a formatted error block explaining the failure. If network connectivity is lost, a persistent banner appears at the top of every page warning the user that they are offline. The app retries any pending requests once the connection returns and replaces the banner with a brief "Reconnected" alert.

Attempting to navigate to a restricted route, for example if the user is not an administrator and tries to access an admin-only CLI task, the server responds with a 403 status. The front end captures this and shows a friendly message that the user does not have the necessary permissions, offering a button to return to the Home view.

## Conclusion and Overall App Journey

Starting from the landing page, users create an account or sign in, recover forgotten passwords, and arrive at a feature-rich dashboard. From there, they switch seamlessly between a powerful code editor and a real-time terminal for executing CLI commands. They manage personal details, theme preferences, and notifications in Settings before returning to daily developer tasks. The app gracefully handles errors, lost connections, and permission issues, guiding users back into the flow. Typical end goals include editing code, running commands in sandboxed containers, and viewing streamed outputs, all within a unified, secure, and responsive interface.