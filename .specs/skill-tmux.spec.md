# Tmux Skill Specification

## Intent

Remote-control tmux terminal multiplexer sessions for interactive CLI automation by sending keystrokes and scraping pane output. Enables automation of terminal applications and interactive command-line workflows.

## Interface

### Core Operations
- **Session Management**: Create, attach, detach, and list tmux sessions
- **Pane Control**: Send commands and keystrokes to specific panes
- **Output Capture**: Scrape and monitor pane content and output
- **Window Management**: Control windows and pane layouts
- **Interactive Automation**: Automate terminal applications requiring user input

### Command Structure
- **Session Control**: `tmux new-session`, `tmux attach-session`
- **Send Commands**: `tmux send-keys -t session:window.pane "command" Enter`
- **Capture Output**: `tmux capture-pane -t session:window.pane -p`
- **Window Management**: `tmux new-window`, `tmux split-window`

## Business Logic

### Session and Window Management
1. Create and manage tmux sessions for different workflows
2. Handle window creation and pane splitting for complex layouts
3. Process session naming and organization strategies
4. Manage session persistence and recovery

### Command Execution and Automation
1. Send keystrokes and commands to specific tmux panes
2. Handle command timing and synchronization
3. Process interactive application workflows
4. Manage command history and session state

### Output Monitoring and Scraping
1. Capture pane content for analysis and processing
2. Monitor output changes and trigger conditions
3. Parse command output and extract relevant information
4. Handle scrollback and history management

## Data Structures

### Tmux Session
```typescript
interface TmuxSession {
  name: string; // session name
  id: string; // session ID
  windows: TmuxWindow[]; // session windows
  attached: boolean; // attachment status
  created: Date; // creation time
  activity: Date; // last activity
}

interface TmuxWindow {
  id: number; // window index
  name: string; // window name
  panes: TmuxPane[]; // window panes
  active: boolean; // active window flag
  layout: string; // pane layout
}
```

### Pane Information
```typescript
interface TmuxPane {
  id: number; // pane index
  active: boolean; // active pane flag
  dimensions: { width: number; height: number };
  cursor: { x: number; y: number }; // cursor position
  title: string; // pane title
  pid: number; // process ID
}
```

### Command Execution
```typescript
interface PaneCommand {
  target: string; // session:window.pane target
  command: string; // command to send
  keys: string[]; // additional keys (Enter, C-c, etc.)
  timing?: number; // delay between keystrokes
}

interface OutputCapture {
  target: string; // pane target
  content: string; // captured text
  timestamp: Date; // capture time
  scrollback: boolean; // include scrollback
  startLine?: number; // start line number
  endLine?: number; // end line number
}
```

## Constraints

### Platform Requirements
- **Unix Systems**: tmux available on macOS and Linux only
- **Terminal Environment**: Requires terminal multiplexer support
- **Process Management**: Needs process control capabilities
- **Session Persistence**: Depends on tmux server availability

### Interactive Application Limits
- **Timing Sensitivity**: Interactive apps may require specific timing
- **Screen Dependencies**: Some applications require specific terminal sizes
- **Input Complexity**: Limited support for complex input sequences
- **State Management**: Difficult to track application internal state

## Edge Cases

### Session and Process Issues
- **Tmux Server Down**: tmux server not running or crashed
- **Session Not Found**: Referenced sessions no longer exist
- **Permission Issues**: Insufficient permissions for tmux operations
- **Process Conflicts**: Multiple processes competing for pane input

### Command and Timing Problems
- **Command Timing**: Commands sent too quickly causing issues
- **Output Buffering**: Output not immediately available for capture
- **Screen Size**: Applications failing due to terminal size constraints
- **Input Blocking**: Interactive applications waiting for specific input

## Technical Debt

### Limited Error Handling
- Basic error detection for failed tmux operations
- No sophisticated retry mechanisms for transient failures
- Limited debugging capabilities for complex automation scenarios

### Automation Complexity
- Manual timing management for interactive applications
- No built-in application state detection or synchronization
- Complex command sequencing without high-level abstractions

## Dependencies

### Core Dependencies
- **tmux**: Terminal multiplexer installation
- **Shell Environment**: Unix shell for command execution
- **Process Control**: System process management capabilities
- **Terminal Support**: Terminal emulation and control sequences

### System Dependencies
- **Operating System**: macOS or Linux platform support
- **Terminal Emulator**: Compatible terminal environment
- **Process Management**: System-level process control
- **File Permissions**: Access to tmux socket and session files