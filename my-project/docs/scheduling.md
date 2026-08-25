# 5. Scheduling

The round lifecycle — a round opening and closing without any manual action — is
scheduled using **node-cron**, running inside the Express process and checking
round times at a fixed interval. This requires no additional infrastructure, which
is appropriate for the Basic tier's scope.

The scheduling logic is isolated in its own module so that it can later be replaced
with **BullMQ and Redis**, which fire jobs at an exact scheduled time rather than on
a polling interval, once the Advanced tier's requirement for a single authoritative
clock and exactly-once submission handling under load is taken on. Isolating the
scheduling mechanism behind this module boundary means that change will not affect
the rest of the codebase.