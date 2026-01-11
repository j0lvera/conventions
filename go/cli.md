# Go CLI Application Conventions

Conventions specific to building command-line interface applications with Go, Cobra, and Viper.

## Quick Start

### Scaffold the Project

```bash
mkdir cli-project
cd cli-project
go mod init github.com/j0lvera/cli-project
touch main.go
```

### Add Dependencies

```bash
go get github.com/spf13/cobra
go get github.com/spf13/viper
go get github.com/kelseyhightower/envconfig
```

## Project Structure

### Standard CLI Directory Layout

```
cli-project/
├── cmd/
│   └── main/           # Main application entry point
│       └── main.go
├── internal/
│   ├── cli/           # CLI-specific code
│   ├── config/        # Configuration management
│   └── commands/      # Command implementations
├── pkg/               # Public library code
├── go.mod
└── go.sum
```

## Configuration Management

### Configuration with envconfig Pattern

```go
// internal/config/config.go
package config

import (
    "time"

    "github.com/kelseyhightower/envconfig"
)

type Config struct {
    Env             string        `envconfig:"ENV" default:"dev"`             // Default to development environment
    Port            string        `envconfig:"PORT" default:"8080"`           // Default port for the application
    DatabaseURL     string        `envconfig:"DATABASE_URL"`                  // Database connection URL
    RequestTimeout  time.Duration `envconfig:"REQUEST_TIMEOUT" default:"30s"` // Global request timeout
    DatabaseTimeout time.Duration `envconfig:"DATABASE_TIMEOUT" default:"5s"` // Database operation timeout
    Origins         []string      `envconfig:"ORIGINS" default:"http://localhost:3000"` // CORS allowed origins
    DefaultLimit    int           `envconfig:"DEFAULT_LIMIT" default:"100"`   // Default limit for list endpoints
    
    // CLI-specific configuration
    Verbose  bool   `envconfig:"VERBOSE" default:"false"`
    Output   string `envconfig:"OUTPUT" default:"text"`
    LogLevel string `envconfig:"LOG_LEVEL" default:"info"`
}

// LoadEnv loads the configuration from environment variables
func (c Config) LoadEnv() (Config, error) {
    cfg := c

    // load environment variables into the Config struct
    if err := envconfig.Process("", &cfg); err != nil {
        // if there is an error, return the default config and the error
        return c, err
    }

    // return the loaded config
    return cfg, nil
}

func New() (*Config, error) {
    var cfg Config
    loadedCfg, err := cfg.LoadEnv()
    if err != nil {
        return nil, err
    }
    return &loadedCfg, nil
}
```

## Command Structure with Cobra + Viper

### Root Command Setup

```go
// internal/cli/root.go
package cli

import (
    "fmt"
    "os"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"
    "your-project/internal/config"
)

type CLI struct {
    config *config.Config
}

func NewCLI(cfg *config.Config) *CLI {
    return &CLI{config: cfg}
}

func (c *CLI) RootCommand() *cobra.Command {
    rootCmd := &cobra.Command{
        Use:   "myapp",
        Short: "A brief description of your application",
        Long: `A longer description that spans multiple lines and likely contains
examples and usage of using your application.`,
        PersistentPreRun: func(cmd *cobra.Command, args []string) {
            // Bind viper to cobra flags
            viper.BindPFlags(cmd.Flags())
        },
    }

    // Global flags
    rootCmd.PersistentFlags().BoolP("verbose", "v", false, "Enable verbose output")
    rootCmd.PersistentFlags().StringP("output", "o", "text", "Output format (text, json, yaml)")
    rootCmd.PersistentFlags().String("log-level", "info", "Log level (debug, info, warn, error)")

    // Bind flags to viper
    viper.BindPFlag("verbose", rootCmd.PersistentFlags().Lookup("verbose"))
    viper.BindPFlag("output", rootCmd.PersistentFlags().Lookup("output"))
    viper.BindPFlag("log-level", rootCmd.PersistentFlags().Lookup("log-level"))

    // Add subcommands
    rootCmd.AddCommand(c.createCommand())
    rootCmd.AddCommand(c.listCommand())
    rootCmd.AddCommand(c.deleteCommand())

    return rootCmd
}

func (c *CLI) Execute() error {
    return c.RootCommand().Execute()
}
```

### Main Entry Point

```go
// cmd/main/main.go
package main

import (
    "fmt"
    "os"

    "your-project/internal/cli"
    "your-project/internal/config"
)

func main() {
    // Load configuration
    cfg, err := config.New()
    if err != nil {
        fmt.Printf("Failed to load configuration: %v\n", err)
        os.Exit(1)
    }

    // Create and execute CLI
    cliApp := cli.NewCLI(cfg)
    if err := cliApp.Execute(); err != nil {
        fmt.Printf("Command execution failed: %v\n", err)
        os.Exit(1)
    }
}
```

## Command Patterns

### Subcommand Implementation

```go
// internal/cli/create.go
func (c *CLI) createCommand() *cobra.Command {
    cmd := &cobra.Command{
        Use:   "create [name]",
        Short: "Create a new resource",
        Long:  `Create a new resource with the specified name and optional description.`,
        Args:  cobra.ExactArgs(1),
        RunE: func(cmd *cobra.Command, args []string) error {
            return c.runCreate(cmd, args)
        },
    }

    // Command-specific flags
    cmd.Flags().StringP("description", "d", "", "Description of the resource")
    cmd.Flags().BoolP("force", "f", false, "Force creation even if resource exists")

    return cmd
}

func (c *CLI) runCreate(cmd *cobra.Command, args []string) error {
    name := args[0]
    description, _ := cmd.Flags().GetString("description")
    force, _ := cmd.Flags().GetBool("force")
    verbose := viper.GetBool("verbose")

    if verbose {
        fmt.Printf("Creating resource: %s\n", name)
        if description != "" {
            fmt.Printf("Description: %s\n", description)
        }
    }

    // Implementation logic here
    result, err := c.createResource(name, description, force)
    if err != nil {
        return fmt.Errorf("failed to create resource: %w", err)
    }

    return c.outputResult(result)
}
```

### List Command Pattern

```go
// internal/cli/list.go
func (c *CLI) listCommand() *cobra.Command {
    cmd := &cobra.Command{
        Use:   "list",
        Short: "List all resources",
        Long:  `List all resources with optional filtering and pagination.`,
        RunE: func(cmd *cobra.Command, args []string) error {
            return c.runList(cmd, args)
        },
    }

    cmd.Flags().StringP("filter", "f", "", "Filter resources by name pattern")
    cmd.Flags().IntP("limit", "l", c.config.DefaultLimit, "Maximum number of resources to return")
    cmd.Flags().IntP("offset", "s", 0, "Number of resources to skip")

    return cmd
}

func (c *CLI) runList(cmd *cobra.Command, args []string) error {
    filter, _ := cmd.Flags().GetString("filter")
    limit, _ := cmd.Flags().GetInt("limit")
    offset, _ := cmd.Flags().GetInt("offset")

    resources, err := c.listResources(filter, limit, offset)
    if err != nil {
        return fmt.Errorf("failed to list resources: %w", err)
    }

    return c.outputResult(resources)
}
```

## Error Handling

### CLI Error Patterns

```go
func runCommand() error {
    if err := validateInput(); err != nil {
        return fmt.Errorf("validation failed: %w", err)
    }
    
    if err := performAction(); err != nil {
        return fmt.Errorf("action failed: %w", err)
    }
    
    return nil
}
```

## Output Formatting

### Multiple Output Formats with Viper

```go
// internal/cli/output.go
package cli

import (
    "encoding/json"
    "fmt"
    "os"
    "text/tabwriter"

    "github.com/spf13/viper"
    "gopkg.in/yaml.v3"
)

type OutputFormatter interface {
    Format(data interface{}) error
}

type JSONFormatter struct{}
type TableFormatter struct{}
type YAMLFormatter struct{}

func (j JSONFormatter) Format(data interface{}) error {
    output, err := json.MarshalIndent(data, "", "  ")
    if err != nil {
        return err
    }
    fmt.Println(string(output))
    return nil
}

func (y YAMLFormatter) Format(data interface{}) error {
    output, err := yaml.Marshal(data)
    if err != nil {
        return err
    }
    fmt.Print(string(output))
    return nil
}

func (t TableFormatter) Format(data interface{}) error {
    w := tabwriter.NewWriter(os.Stdout, 0, 0, 2, ' ', 0)
    defer w.Flush()

    // Type assertion and table formatting logic
    switch v := data.(type) {
    case []Resource:
        fmt.Fprintln(w, "NAME\tDESCRIPTION\tCREATED")
        for _, resource := range v {
            fmt.Fprintf(w, "%s\t%s\t%s\n", resource.Name, resource.Description, resource.CreatedAt)
        }
    default:
        return fmt.Errorf("unsupported data type for table format")
    }
    return nil
}

func (c *CLI) outputResult(data interface{}) error {
    format := viper.GetString("output")
    
    var formatter OutputFormatter
    switch format {
    case "json":
        formatter = JSONFormatter{}
    case "yaml":
        formatter = YAMLFormatter{}
    case "table", "text":
        formatter = TableFormatter{}
    default:
        return fmt.Errorf("unsupported output format: %s", format)
    }

    return formatter.Format(data)
}
```

## Logging for CLI

### CLI-appropriate Logging with Viper Integration

```go
// internal/cli/logging.go
package cli

import (
    "os"

    "github.com/rs/zerolog"
    "github.com/spf13/viper"
)

func setupLogger() zerolog.Logger {
    logLevel := viper.GetString("log-level")
    verbose := viper.GetBool("verbose")

    // Set log level based on configuration
    level := zerolog.InfoLevel
    switch logLevel {
    case "debug":
        level = zerolog.DebugLevel
    case "warn":
        level = zerolog.WarnLevel
    case "error":
        level = zerolog.ErrorLevel
    }

    if verbose {
        level = zerolog.DebugLevel
    }

    // Configure logger for CLI output
    logger := zerolog.New(os.Stderr).With().Timestamp().Logger().Level(level)
    
    // Use console writer for better CLI experience
    if level == zerolog.DebugLevel {
        logger = logger.Output(zerolog.ConsoleWriter{Out: os.Stderr})
    }

    return logger
}

// Usage in commands
func (c *CLI) runCreate(cmd *cobra.Command, args []string) error {
    logger := setupLogger()
    
    logger.Info().Msg("Processing files...")  // User-facing
    logger.Debug().Str("file", filename).Msg("Processing file")  // Debug only
    
    return nil
}
```

## Testing CLI Applications

### Command Testing

```go
// internal/cli/create_test.go
package cli

import (
    "bytes"
    "testing"

    "github.com/spf13/viper"
    "github.com/stretchr/testify/assert"
    "your-project/internal/config"
)

func TestCreateCommand(t *testing.T) {
    // Setup test configuration
    cfg := &config.Config{
        Env:          "test",
        DefaultLimit: 10,
        Verbose:      true,
        Output:       "json",
    }
    
    cli := NewCLI(cfg)
    
    // Capture output
    var buf bytes.Buffer
    cmd := cli.createCommand()
    cmd.SetOut(&buf)
    cmd.SetErr(&buf)
    
    // Set command arguments
    cmd.SetArgs([]string{"test-resource", "--description", "Test description"})
    
    // Execute command
    err := cmd.Execute()
    assert.NoError(t, err)
    
    // Verify output
    output := buf.String()
    assert.Contains(t, output, "test-resource")
}

func TestListCommandWithFlags(t *testing.T) {
    cfg := &config.Config{DefaultLimit: 100}
    cli := NewCLI(cfg)
    
    cmd := cli.listCommand()
    cmd.SetArgs([]string{"--filter", "test", "--limit", "50"})
    
    err := cmd.Execute()
    assert.NoError(t, err)
}

// Integration test with viper
func TestCommandWithViperConfig(t *testing.T) {
    // Setup viper for testing
    viper.Set("verbose", true)
    viper.Set("output", "json")
    
    cfg := &config.Config{}
    cli := NewCLI(cfg)
    
    cmd := cli.RootCommand()
    cmd.SetArgs([]string{"create", "test-resource", "--verbose"})
    
    err := cmd.Execute()
    assert.NoError(t, err)
    
    // Cleanup
    viper.Reset()
}
```

## Best Practices

### Command Design
- Use consistent flag naming across commands (`--verbose`, `--output`, `--force`)
- Provide both short and long flag options (`-v`/`--verbose`, `-o`/`--output`)
- Include helpful examples in command descriptions
- Use cobra's built-in validation (`cobra.ExactArgs(1)`, `MarkFlagRequired()`)
- Group related flags logically

### Configuration Management
- Use envconfig pattern for consistent environment variable handling
- Combine Viper for flag binding with envconfig for environment loading
- Provide sensible defaults for all configuration options
- Document all environment variables and their defaults

### Output and User Experience
- Support multiple output formats (text, json, yaml)
- Use structured logging with appropriate levels
- Implement proper signal handling for long-running operations
- Use progress bars for long operations
- Provide machine-readable output formats for automation
- Follow UNIX conventions for exit codes (0 = success, 1 = error)

### Error Handling
- Use `RunE` instead of `Run` for commands that can fail
- Wrap errors with context using `fmt.Errorf("operation failed: %w", err)`
- Provide helpful error messages with suggestions
- Log errors appropriately (user-facing vs debug information)

### Testing
- Test commands with various flag combinations
- Use simple constructor injection for testable CLI components
- Mock external dependencies in tests
- Test output formatting for different formats
- Verify error handling and exit codes

### Security
- Validate all user inputs
- Use secure defaults for configuration
- Don't log sensitive information
- Handle file permissions appropriately