# Parallel Programming in Java 

## Week 1


### Task Creation and Termination 

 The async notation for task creation: “async ⟨stmt1⟩”, causes the parent task (i.e., the task executing the async statement) to create a new child task to execute the body of the async, ⟨stmt1⟩, asynchronously (i.e., before, after, or in parallel) with the remainder of the parent task. We also learned the finish notation for task termination: “finish ⟨stmt2⟩” causes the parent task to execute ⟨stmt2⟩, and then wait until ⟨stmt2⟩ and all async tasks created within ⟨stmt2⟩ have completed. Async and finish constructs may be arbitrarily nested.

 ```
 finish {
  async S1; // asynchronously compute sum of the lower half of the array
  S2;       // compute sum of the upper half of the array in parallel with S1
}
S3; // combine the two partial sums after both S1 and S2 have finished
 ```

 While async and finish notations are useful algorithmic/pseudocode notations, we also provide you access to a high-level open-source Java-8 library called PCDP (for Parallel, Concurrent, and Distributed Programming), for which the source code is available at [PCDP](https://github.com/habanero-rice/pcdp). PCDP contains APIs (application programming interfaces) that directly support async and finish constructs so that you can use them in real code as well. In the next lecture, you will learn how to to implement the async and finish functionality using Java’s standard Fork/Join (FJ) framework.


In the Fork Join framework, a task can be specified in the {\tt compute()}compute() method of a user-defined class that extends the standard RecursiveAction class in the FJ framework. In our Array Sum example, we created class {\tt ASum}ASum with fields {\tt A}A for the input array, {\tt LO}LO and {\tt HI}HI for the subrange for which the sum is to be computed, and {\tt SUM}SUM for the result for that subrange. For an instance of this user-defined class (e.g., {\tt L}L in the lecture), we learned that the method call, {\tt L.fork()}L.fork(), creates a new task that executes {\tt L}L’s {\tt compute()}compute() method. This implements the functionality of the async construct that we learned earlier. The call to {\tt L.join()}L.join() then waits until the computation created by {\tt L.fork()}L.fork() has completed. Note that {\tt join()}join() is a lower-level primitive than finish because {\tt join()}join() waits for a specific task, whereas finish implicitly waits for all tasks created in its scope. To implement the finish construct using {\tt join()}join() operations, you have to be sure to call {\tt join()}join() on every task created in the finish scope.

FJ tasks are executed in a ForkJoinPool, which is a pool of Java threads. This pool supports the invokeAll() method that combines both the {\tt fork}fork and {\tt join}join operations by executing a set of tasks in parallel, and waiting for their completion. For example, {\tt ForkJoinTask.invokeAll(left, right)}ForkJoinTask.invokeAll(left,right) implicitly performs {\tt fork()}fork() operations on {\tt left}left and {\tt right}right, followed by {\tt join()}join() operations on both objects.

Computation Graphs (CGs), which model the execution of a parallel program as a partially ordered set. Specifically, a CG consists of:

A set of vertices or nodes, in which each node represents a step consisting of an arbitrary sequential computation.
A set of directed edges that represent ordering constraints among steps.
For fork–join programs, it is useful to partition the edges into three cases:

1. Continue edges that capture sequencing of steps within a task.

2. Fork edges that connect a fork operation to the first step of child tasks.

3. Join edges that connect the last step of a task to all join operations on that task.

CGs can be used to define data races, an important class of bugs in parallel programs. We say that a data race occurs on location L in a computation graph, G, if there exist steps S1 and S2 in G such that there is no path of directed edges from S1 to S2 or from S2 to S1 in G, and both S1 and S2 read or write L (with at least one of the accesses being a write, since two parallel reads do not pose a problem).

CGs can also be used to reason about the ideal parallelism of a parallel program as follows:

Define WORK(G) to be the sum of the execution times of all nodes in CG G,
Define SPAN(G) to be the length of a longest path in G, when adding up the execution times of all nodes in the path. The longest paths are known as critical paths, so SPAN also represents the critical path length (CPL) of G.
Given the above definitions of WORK and SPAN, we define the ideal parallelism of Computation Graph G as the ratio, WORK(G)/SPAN(G). The ideal parallelism is an upper limit on the speedup factor that can be obtained from parallel execution of nodes in computation graph G. Note that ideal parallelism is only a function of the parallel program, and does not depend on the actual parallelism available in a physical computer.

the possible executions of a Computation Graph (CG) on an idealized parallel machine with P processors. It is idealized because all processors are assumed to be identical, and the execution time of a node is assumed to be fixed, regardless of which processor it executes on. A legal schedule is one that obeys the dependence constraints in the CG, such that for every directed edge (A, B), the schedule guarantees that step B is only scheduled after step A completes. Unless otherwise specified, we will restrict our attention in this course to schedules that have no unforced idleness, i.e., schedules in which a processor is not permitted to be idle if a CG node is available to be scheduled on it. Such schedules are also referred to as "greedy" schedules.


We defined T_{P}   as the execution time of a CG on P processors, and observed that

T_{inf} <= T_{P} <= T_{1}

We also saw examples for which there could be different values of T_{P} for different schedules of the same CG on P processors.

We then defined the parallel speedup for a given schedule of a CG on P processors as Speedup(P) = T_{1}
​, and observed that Speedup(P) must be ≤ the number of processors P , and also ≤ the ideal parallelism, WORK/SPAN.

a simple observation made by Gene Amdahl in 1967: if q ≤ 1 is the fraction of WORK in a parallel program that must be executed sequentially, then the best speedup that can be obtained for that program for any number of processors, P , is Speedup(P) ≤ 1/q.

This observation follows directly from a lower bound on parallel execution time that you are familiar with, namely T_P  ≥ SPAN(G). If fraction q of WORK(G) is sequential, it must be the case that SPAN(G) ≥ q × WORK(G). Therefore, Speedup(P) = T_{1}  /T_{P} must be ≤ WORK(G)/(q × WORK(G)) = 1/q since T_{1} = WORK(G) for greedy schedulers.

Amdahl’s Law reminds us to watch out for sequential bottlenecks both when designing parallel algorithms and when implementing programs on real machines. As an example, if q = 10%, then Amdahl's Law reminds us that the best possible speedup must be ≤ 10 (which equals 1/q ), regardless of the number of processors available.
