---
layout: post
title: Locks should be used carefully 
---

Using locks in parallel processing can make it twice slower than single threaded. So sometimes it could be better to use plain old single thread approach. 

<!--more-->

### The code
```csharp
    class Program
    {
        private static readonly string _filePath = @"C:\access.log";
        private static object _syncRoot = new object();


        public static void TestSpeedParallelForConcurrencyBag()
        {
            var allLines = File.ReadAllLines(_filePath);
            int linesToProcess = allLines.Length;
            var parsedLogEntries = new ConcurrentBag<ApacheLogEntry>();

            Stopwatch sw = new Stopwatch();
            sw.Start();

            Parallel.For(0, linesToProcess, i => parsedLogEntries.Add(ApacheLogAnalyser.GetLogEntryFromString(allLines[i])));

            sw.Stop();
            Console.WriteLine("TestSpeedParallelForConcurrencyBag() Time: {0} ms, Count: {1}", sw.Elapsed.TotalMilliseconds, parsedLogEntries.Count);
        }

        public static void TestSpeedParallelForList()
        {
            var allLines = File.ReadAllLines(_filePath);
            int linesToProcess = allLines.Length;
            var parsedLogEntries = new List<ApacheLogEntry>();

            Stopwatch sw = new Stopwatch();
            sw.Start();

            Parallel.For(0, linesToProcess, i =>
            {
                lock (_syncRoot)
                {
                    parsedLogEntries.Add(ApacheLogAnalyser.GetLogEntryFromString(allLines[i]));
                }
            });

            sw.Stop();
            Console.WriteLine("TestSpeedParallelForList() Time: {0} ms, Count: {1}", sw.Elapsed.TotalMilliseconds, parsedLogEntries.Count);
        }

        public static void TestSpeedForeachList()
        {
            var allLines = File.ReadAllLines(_filePath);
            var parsedLogEntries = new List<ApacheLogEntry>();

            Stopwatch sw = new Stopwatch();
            sw.Start();

            foreach (var line in allLines)
            {
                parsedLogEntries.Add(ApacheLogAnalyser.GetLogEntryFromString(line));
            }


            sw.Stop();
            Console.WriteLine("TestSpeedForeachList() Time: {0} ms, Count: {1}", sw.Elapsed.TotalMilliseconds, parsedLogEntries.Count);
        }


        static void Main(string[] args)
        {
            TestSpeedForeachList();
            TestSpeedParallelForList();
            TestSpeedParallelForConcurrencyBag();

            Console.ReadKey();
        }
    }
```
### Results

TestSpeedForeachList() Time: 6991.1298 ms, Count: 501366

TestSpeedParallelForList() Time: 11022.504 ms, Count: 501366

TestSpeedParallelForConcurrencyBag() Time: 5173.9488 ms, Count: 501366