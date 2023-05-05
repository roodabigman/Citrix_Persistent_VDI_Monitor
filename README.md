# Persistent_VDI_Monitor
PoSH scripts for collecting data from Monitor API to determine assigned VDI usage metrics

Citrix customers with persistent VDI use cases have a blind spot to the usage of those machines.  Things like Monitor / Analytics are focused on usage tracking delivery group / site wide, or drilling down to a single user, but dont provide an easy way to see individual usage metrics for all users in the site.  Many customers have used broker powershell SDK commands to retrieve most recent usage statistics, but this is only half the issue, if someone accesses their VDI every day, but is only connected for 5 minutes a day, they show as using the machine frequently, but they dont use the machine a lot, and the use case should probably be revisited.

This script utlizes the Monitor API to collect usage data for assigned machines within a given timeframe.  Default setting is 7 day "lookback period" i.e. how much have these machines been used in the past week.

For each machine, we calculate the "last usage" timestamp (most recent connection disconnect from the machine), and total usage time (sum of connected time for all connections in the past week), and write the results to an array of PSCustomObjects that is then written out to CSV.

Note about the data format: I often get asked about "sessions" vs "connections".  in a persistent VDI model a session is often a very long running element, as if the machine is not power managed and a user disconnects instead of logs off at the end of the day, the session persists until the next morning when the user connects again.  this is how you get sessions with multiple associated connections.  For total usage time and last used timestamp the connection is what we care about, even if the session is still active, the user is not connected to the VDI, so we dont want to consider that in our last or aggregate usage.

So why are we collecting Session data?  a few reasons...
1. we have to get to connections. the format of the data in the Monitor DB is such that if we search on Machines, machines have an associated sessions table we have to expand to look at sessions, which has an associated Connections table we have to expand to look at associated connections.  so to get to connections, we have to expand sessions

2. session data is useful for data validation.  failed logons to a VDI show up as a session in the DB, but the start and end times get the same timestamp, this is a useful check for us to know when to ignore sessions.  Also, sometimes connections have bad data.... disconnect times will be missing for connections that are no longer active.  Looking at a session end date helps us determine if the connection missing an enddate is because the sessions is truly active or its just erroneous data in the DB

3. If you have disconnected timers set that log off the user after a period of being disconnected the sessions data / timing becomes more interesting, but the script doesnt analyze anything related to that, yet :)
