# su26-ai301-contribution

## Contribution 1: Add Synthetic Endpoint Probing and Active Status Checks

* **Contribution Number:** 1
* **Student:** Mohammadarshya Salehibakhsh
* **Issue:** https://github.com/SikamikanikoBG/homelab-monitor/issues/161
* **Status:** Phase I Complete

---

## Why I Chose This Issue

I chose to work on this feature because it bridges a critical gap in lightweight infrastructure observability—moving from process-level health tracking to functional network verification. In my previous full-stack web development projects, I built data loops and interfaces using Python and Flask, but I have not yet written an isolated background task scheduler or native socket-level probing scripts from scratch. This contribution directly aligns with my goal to strengthen my systems-level engineering and backend design skills.

Furthermore, the issue is exceptionally well-scoped for a 3–4 week development lifecycle. The project maintainer laid out explicit architectural boundaries: avoiding external third-party dependencies by sticking strictly to Python's standard libraries (`urllib.request` and `socket`), using an isolated local SQLite history database schema, and creating predictable REST API endpoints. This clean segregation of concerns ensures I can build, verify, and deliver a robust production-style feature without risking severe scope creep.

---

## Understanding the Issue

### Problem Description
The `homelab-monitor` platform currently tracks whether backend services, host systems, or containers are running by looking purely at process lifecycle and container states. However, it completely lacks verification at the application layer. This introduces a significant blind spot where a local container or systemd unit could be technically marked as "Up" or active, while the underlying web server or application endpoint is frozen, misconfigured, or returning error codes (e.g., a site crashing with an internal server error or timing out). 

### Expected Behavior
The platform should support an active "Checks" subsystem that allows users to routinely probe endpoints on a configurable schedule. It must evaluate physical connectivity across three core protocols:
* **HTTP(s):** Verifies that the endpoint returns expected status codes (e.g., 2xx/3xx), stays within a given response-time threshold, and matches optional body strings.
* **TCP:** Validates port reachability by actively opening a network socket connection.
* **Ping:** Checks raw ICMP network host reachability, dropping back gracefully to TCP connection attempts if root execution privileges are missing.

All administrative mutations (Create, Read, Update, Delete) must expose safe CRUD endpoints via `/api/checks`, store active execution data over a thread loop inside SQLite, and dynamically surface an active overview layout inside a custom web tab.

### Current Behavior
Currently, the system is entirely passive. It reads container and process states using internal host mechanisms but contains no worker threads, schedulers, or outbound testing probers to verify live endpoint network responsiveness. 

### Affected Components
* **Backend (`app.py` / database layers):** Needs a new `checks` SQLite table layout, a persistent polling/scheduler thread routine to evaluate targets at given intervals, and corresponding CRUD API path routers.
* **Frontend (`static/dashboard.html` / UI controllers):** Requires a brand new dashboard view pane displaying an aggregate health status layout (live red/green visual state dots), latency trends, and state management input configuration windows.
