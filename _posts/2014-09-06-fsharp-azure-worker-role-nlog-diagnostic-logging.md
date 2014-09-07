---
title: F# Azure Worker Role NLog Logging Setup
layout: post
excerpt: Example code to get diagnostic logging set up for a worker role via NLog and Azure Table Storage.
---

# F# Azure Worker Role NLog Logging Setup

I didn't see any explicit examples of this online so I thought I'd post mine.

Originally Microsoft posted "recommended" diagnostic setup code for azure worker roles so that trace events would be logged to Azure table storage. I'm not entirely sure if that is still the case, but here is the F# code we use to get the trace sent to table storage.

Note that all of our logging we do via NLog-- its setup is in here as well.

{% highlight fsharp %}

namespace YourNamespace

open System
open System.Collections.Generic
open System.Diagnostics
open System.Linq
open System.Net
open System.Threading
open Microsoft.WindowsAzure
open Microsoft.WindowsAzure.Diagnostics;
open Microsoft.WindowsAzure.ServiceRuntime;
open Microsoft.WindowsAzure.Diagnostics.Management;

open NLog
open NLog.Config
open NLog.Common
open NLog.Targets

type WorkerRole() =
    inherit RoleEntryPoint() 

    // generic msft log fn, also pubs to azure table storage
    let log message (kind : string) = Trace.TraceInformation(message, kind)

    override wr.Run() =
        // put your work here
        ()

    override wr.OnStart() = 
        // Set the maximum number of concurrent connections 
        ServicePointManager.DefaultConnectionLimit <- 12
       
        // For information on handling configuration changes
        // see the MSDN topic at http://go.microsoft.com/fwlink/?LinkId=166357.

        //
        // this is great for debugging--
        // will write messages to your VS Output window
        //
        let conf = new LoggingConfiguration()
        let debug = new DebugTarget()
        conf.AddTarget("debug",debug)
        let debugLayout = new Layouts.SimpleLayout("${longdate}|${machinename}|[${threadid}]|${level:uppercase=true}|${logger}|${message}|${exception:format=ToString,StackTrace}")
        debug.Layout <- debugLayout
        let debugRule = new LoggingRule("*", NLog.LogLevel.Debug, debug)
        conf.LoggingRules.Add(debugRule)

        //
        // have nlog publish to the azure table storage
        // by having it publish to trace
        //
        let trace = new TraceTarget()
        conf.AddTarget("trace",trace)
        trace.Layout <- debugLayout
        let traceRule = new LoggingRule("*", NLog.LogLevel.Debug, trace)
        conf.LoggingRules.Add(traceRule)

        //
        // azure publish setup
        //

        let conString = RoleEnvironment.GetConfigurationSettingValue "Microsoft.WindowsAzure.Plugins.Diagnostics.ConnectionString"
        let dm = new RoleInstanceDiagnosticManager(
                    conString,
                    RoleEnvironment.DeploymentId,
                    RoleEnvironment.CurrentRoleInstance.Role.Name,
                    RoleEnvironment.CurrentRoleInstance.Id)

        //get the current configuration
        let mutable dc = dm.GetCurrentConfiguration()

        //if that failed, get the values from config file
        if dc = null then dc <- DiagnosticMonitor.GetDefaultInitialConfiguration()

        //Windows Azure Logs
        dc.Logs.BufferQuotaInMB <- 10
        dc.Logs.ScheduledTransferLogLevelFilter <- Microsoft.WindowsAzure.Diagnostics.LogLevel.Verbose
        dc.Logs.ScheduledTransferPeriod <- TimeSpan.FromMinutes((float)5)

        //Windows Event Logs
        dc.WindowsEventLog.BufferQuotaInMB <- 10
        dc.WindowsEventLog.DataSources.Add("System!*")
        dc.WindowsEventLog.DataSources.Add("Application!*")
        dc.WindowsEventLog.ScheduledTransferPeriod <- TimeSpan.FromMinutes((float)15)

        //Performance Counters
        dc.PerformanceCounters.BufferQuotaInMB <- 10;
        let perfConfig = new PerformanceCounterConfiguration()
        perfConfig.CounterSpecifier <- @"\Processor(_Total)\% Processor Time"
        perfConfig.SampleRate <- System.TimeSpan.FromSeconds(float 60)
        dc.PerformanceCounters.DataSources.Add(perfConfig)
        dc.PerformanceCounters.ScheduledTransferPeriod <- TimeSpan.FromMinutes(float 10)

        //Failed Request Logs
        dc.Directories.BufferQuotaInMB <- 10;
        dc.Directories.ScheduledTransferPeriod <- TimeSpan.FromMinutes(float 30)

        //Infrastructure Logs
        dc.DiagnosticInfrastructureLogs.BufferQuotaInMB <- 10
        dc.DiagnosticInfrastructureLogs.ScheduledTransferLogLevelFilter <- Microsoft.WindowsAzure.Diagnostics.LogLevel.Verbose
        dc.DiagnosticInfrastructureLogs.ScheduledTransferPeriod <- TimeSpan.FromMinutes(float 60)

        //Crash Dumps
        CrashDumps.EnableCollection(true)

        //overall quota; must be larger than the sum of all items
        dc.OverallQuotaInMB <- 5000

        //save the configuration
        dm.SetCurrentConfiguration(dc);

        LogManager.Configuration <- conf
        base.OnStart()

    override wr.OnStop() = 
        // put any dispose/cleanup here
        ()

{% endhighlight %}

That's it!