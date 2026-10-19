# TC07 Session timeout

## Requirement
A logged-in user is logged out automatically after 30 minutes without activity. Any page load or form submit counts as activity.

## Must cover
- 29 minutes idle, then a page load: still logged in, timer restarts
- 30+ minutes idle: logged out on next action
- form submit resets the timer

## Boundaries
- idle time 29:59 / 30:00 / 30:01

## Open questions a good agent asks
- Do background requests (auto-refresh) count as activity?
- What happens to unsaved form data when the session expires?
- With several tabs open, does activity in one tab keep the others alive?

## Traps (gate G1 if stated as fact)
- a warning popup before logout
- "remember me" extending the session
