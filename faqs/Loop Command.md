# The /loop command

The /loop command is a scheduling feature built into Claude Code, Anthropic’s developer-focused command-line tool. It essentially functions as an in-session cron job, allowing you to execute prompts, check statuses, or run maintenance tasks repeatedly at set intervals while your terminal session remains open.  


## Command Syntax
You can invoke the loop feature directly from your Claude Code CLI using the following format:

```bash
/loop 
```


* With an Interval: Specifying a time (e.g., 5m, 30s, 1h) instructs Claude to run your prompt at that exact frequency.
* Self-Paced / Empty Interval: If you omit the interval, Claude will automatically self-pace between iterations.
* Empty Prompt: If you omit the prompt, Claude will default to executing its built-in maintenance routine or custom instructions defined inside a local loop.md configuration file.  


## Common Examples


* Monitoring Deploys: /loop 5m check if the deploy finished
* Fixing Build Failures: /loop 45s check the build status from build.log and fix errors
* Summarising Work: /loop every morning use the Slack MCP to give me a summary of top posts I was tagged in  


## Key Constraints & Behaviors


* ⏳ Session-Bound: Loops are tied strictly to your active terminal session. If your terminal closes, your computer goes to sleep, or the active Claude Code session ends, the loop terminates.  
* 🛑 How to Stop: You can easily stop a pending, self-paced loop cycle by pressing the Esc key.  
* ⏱️ API Cost & Scheduling Safety: Because loops run automatically, API costs can scale quickly if short intervals are set. To prevent sudden API spikes and avoid aggressive rate limits, the underlying scheduler adds an intentional random offset (up to 10%) to the execution timer.  



## **Headless mode** run

Claude Code officially supports a fully non-interactive "headless mode", but it functions differently than the active /loop terminal command.  

Because the /loop command is **designed specifically for an interactive, session-bound terminal interface**, it cannot be run headless directly. Instead, you achieve the exact same headless outcome using the native -p (or --print) flag from your system’s command line.  


### How to Run Headless Tasks

You bypass the interactive terminal UI entirely by feeding your task into the -p parameter. Claude will spin up, execute the task to completion, print the final result to stdout, and immediately exit.  

```bash
claude -p "Find and fix the bug in auth.py" --allowedTools "Read,Edit,Bash"
```

(Note: To make it truly autonomous without waiting for your keystrokes, you will often combine it with --allowedTools or the --dangerously-skip-permissions flag so it auto-approves file and bash actions).  


### Simulating a "Headless /loop"

To recreate the periodic loop behavior without keeping an active terminal session open, developers rely on two primary methods:

#### 1. System Cron Jobs (The True Headless Way)

Since headless mode acts like a standard Unix utility, you can offload the scheduling to your operating system using a standard system cron job.  

```bash
# Open your crontab
crontab -e
# Run headless Claude Code every 15 minutes to check build logs
*/15 * * * * cd /path/to/project && claude -p "Check build.log for new errors and fix them" --allowedTools "Read,Edit,Bash"
```

#### 2. Native System Scoped Cron (CronCreate)

If you want to manage schedules inside Claude without system crontabs, you can use Claude's built-in CronCreate tool. In an interactive session, you can instruct Claude to build a background task using standard 5-field cron syntax:  


* Command: what scheduled tasks do I have? or schedule a task to check deploys every 10 minutes
* How it works: Claude utilizes underlying system tools like CronCreate and CronList to manage these background routines.  


**Warning on native loops/cron**: These internal tasks are session-scoped and expire after 7 days, meaning they still rely on Claude Code running and being idle to trigger. For deep production automation or CI/CD pipelines, triggering claude -p via GitHub Actions or your own system-level cron is the recommended headless practice.  


## Running Claude on schedule in containers or k8s pods

Utilizing the native Kubernetes CronJob controller to spin up an ephemeral container triggering Claude Code via claude -p (headless mode) is the industry-standard recommendation for this architecture.  

Relying on Claude Code's internal interactive features (like /loop or CronCreate) inside a long-running container or pod is an anti-pattern for infrastructure. The standard interactive scheduling commands are session-scoped; if a pod crashes, restarts, or scales, Claude's internal memory of that schedule disappears. Offloading the orchestration to Kubernetes ensures resilience and reliability.  

### The Recommended Architecture

When deploying this into Kubernetes, structure your workload as a standard CronJob manifest rather than a perpetual Deployment pod:  

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: claude-autonomous-task
spec:
  schedule: "0 2 * * *" # Runs daily at 2:00 AM
  concurrencyPolicy: Forbid # Prevents overlapping executions if a task runs long
  jobTemplate:
    spec:
      activeDeadlineSeconds: 1800 # 30 min hard limit to prevent hanging pods
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: claude-agent
            image: your-registry/claude-code-runner:latest
            env:
            - name: ANTHROPIC_API_KEY
              valueFrom:
                secretKeyRef:
                  name: claude-secrets
                  key: api-key
            # The core execution command
            command: 
            args:
            - |
              cd /workspace/your-repo
              claude -p "Scan the logs directory, locate new application exceptions, and suggest fixes in a README.md" \
                     --dangerously-skip-permissions \
                     --no-session-persistence
```


### Production Best Practices for Containers

1. Enforce --dangerously-skip-permissions

Without this flag, the headless process will immediately block and stall indefinitely waiting for interactive human user confirmation on tool usage (like reading or writing files).  

2. Always append --no-session-persistence

By default, Claude Code tries to save local state in ~/.claude/sessions. Because Kubernetes pods are ephemeral and have transient file systems, state will be wiped between runs anyway. Explicitly using --no-session-persistence ensures Claude approaches every scheduled trigger with a clean, consistent, and fully stateless context.  

3. Use a Codebase Mount or Git Clone in an InitContainer

Since Claude Code operates directly on a file repository, the pod needs access to the code. A common pattern is utilizing a K8s initContainer to clone the target repository or pull your project logs into a shared emptyDir volume before the claude container fires.  

