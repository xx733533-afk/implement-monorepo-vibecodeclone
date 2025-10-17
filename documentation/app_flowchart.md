flowchart TD
    Start --> VisitPage
    VisitPage --> SignInPage
    VisitPage --> SignUpPage
    SignInPage --> AuthAPI
    SignUpPage --> AuthAPI
    AuthAPI --> AuthCheck{Auth Success}
    AuthCheck -->|Yes| Dashboard
    AuthCheck -->|No| ErrorPage
    Dashboard --> TerminalUI[Terminal Component]
    TerminalUI --> CLICommand[POST to /cli/run]
    CLICommand --> ServerAPI
    ServerAPI --> PermCheck{Authenticated Request}
    PermCheck -->|Yes| CLIManager
    PermCheck -->|No| Error401
    CLIManager --> SpawnSandbox[Spawn Docker Sandbox]
    SpawnSandbox --> WebSocketServer[WebSocket cli logs]
    WebSocketServer --> TerminalUI
    TerminalUI --> StopButton[Stop Command]
    StopButton --> StopAPI[POST to /cli/stop]
    StopAPI --> TerminateSandbox[Terminate Sandbox Process]