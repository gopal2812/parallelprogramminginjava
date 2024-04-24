# ParallelProgrammingInJava
parallel programming in java

## 1 Task Parallelism
Tasks are the most basic unit of parallel programming. An increasing number of programming languages (including Java and C++) are moving from older thread-based approaches to more modern task-based approaches for parallel programming. task creation, task termination, and the “computation graph” theoretical model for understanding various properties of task-parallel programs. These properties include work, span, ideal parallelism, parallel speedup, and Amdahl’s Law. also the Fork/Join framework.
# 1.1 Creating Tasks in Java’s Fork/Join Framework
In this framework, a task can be specified in the 𝚌𝚘𝚖𝚙𝚞𝚝𝚎() method of a user-defined class that extends the standard RecursiveAction class in the FJ framework.
In our Array Sum example, we created class 𝙰𝚂𝚞𝚖 with fields 𝙰 for the input array, 𝙻𝙾 and 𝙷𝙸 for the subrange for which the sum is to be computed, and 𝚂𝚄𝙼 for the result for that subrange. For an instance of this user-defined class (e.g., 𝙻 in the lecture), we learned that the method call, 𝙻.𝚏𝚘𝚛𝚔(), creates a new task that executes 𝙻’s 𝚌𝚘𝚖𝚙𝚞𝚝𝚎() method. This implements the functionality of the async construct that we learned earlier. The call to 𝙻.𝚓𝚘𝚒𝚗() then waits until the computation created by 𝙻.𝚏𝚘𝚛𝚔() has completed. Note that 𝚓𝚘𝚒𝚗() is a lower-level primitive than finish because 𝚓𝚘𝚒𝚗() waits for a specific task, whereas finish implicitly waits for all tasks created in its scope. To implement the finish construct using 𝚓𝚘𝚒𝚗() operations, you have to be sure to call 𝚓𝚘𝚒𝚗() on every task created in the finish scope.
A sketch of the Java code for the ASum class is as follows:
private static class ASum extends RecursiveAction { int[] A; // input array int LO, HI; // subrange int SUM; // return value . . . @Override protected void compute() { SUM = 0; for (int i = LO; i <= HI; i++) SUM += A[i]; } // compute() } 
FJ tasks are executed in a ForkJoinPool, which is a pool of Java threads. This pool supports the invokeAll() method that combines both the 𝚏𝚘𝚛𝚔 and 𝚓𝚘𝚒𝚗 operations by executing a set of tasks in parallel, and waiting for their completion. For example, 𝚒𝚗𝚟𝚘𝚔𝚎𝙰𝚕𝚕(𝚕𝚎𝚏𝚝,𝚛𝚒𝚐𝚑𝚝) implicitly performs 𝚏𝚘𝚛𝚔() operations on 𝚕𝚎𝚏𝚝 and 𝚛𝚒𝚐𝚑𝚝, followed by 𝚓𝚘𝚒𝚗() operations on both objects.
 
# 1.2 Computation Graphs, Work, Span, Ideal Parallelism
 
a CG consists of:
•	A set of vertices or nodes, in which each node represents a step consisting of an arbitrary sequential computation.
•	A set of directed edges that represent ordering constraints among steps.
CGs can be used to define data races, an important class of bugs in parallel programs. We say that a data race occurs on location L in a computation graph, G, if there exist steps S1 and S2 in G such that there is no path of directed edges from S1 to S2 or from S2 to S1 in G, and both S1 and S2 read or write L (with at least one of the accesses being a write, since two parallel reads do not pose a problem).
CGs can also be used to reason about the ideal parallelism of a parallel program as follows:
•	Define WORK(G) to be the sum of the execution times of all nodes in CG G,
•	Define SPAN(G) to be the length of a longest path in G, when adding up the execution times of all nodes in the path. The longest paths are known as critical paths, so SPAN also represents the critical path length (CPL) of G.
Given the above definitions of WORK and SPAN, we define the ideal parallelism of Computation Graph G as the ratio, WORK(G)/SPAN(G). The ideal parallelism is an upper limit on the speedup factor that can be obtained from parallel execution of nodes in computation graph G. Note that ideal parallelism is only a function of the parallel program, and does not depend on the actual parallelism available in a physical computer.
# 1.3 Multiprocessor Scheduling, Parallel Speedup
 
It is idealized because all processors are assumed to be identical, and the execution time of a node is assumed to be fixed, regardless of which processor it executes on. A legal schedule is one that obeys the dependence constraints in the CG, such that for every directed edge (A, B), the schedule guarantees that step B is only scheduled after step A completes. Unless other specified, we will restrict our attention in this course to schedules that have no unforced idleness, i.e., schedules in which a processor is not permitted to be idle if a CG node is available to be scheduled on it. Such schedules are also referred to as “greedy” schedules.
We defined TP as the execution time of a CG on P processors, and observed that
T∞ ≤ TP ≤ T1
We also saw examples for which there could be different values of TP for different schedules of the same CG on P processors.
We then defined the parallel speedup for a given schedule of a CG on P processors as Speedup(P) = T1/TP, and observed that Speedup(P) must be ≤ the number of processors P , and also ≤ the ideal parallelism, WORK/SPAN.
# 1.4 Amdahl’s Law
 
if q ≤ 1 is the fraction of WORK in a parallel program that must be executed sequentially, then the best speedup that can be obtained for that program for any number of processors, P , is Speedup(P) ≤ 1/q.
This observation follows directly from a lower bound on parallel execution time that you are familiar with, namely TP ≥ SPAN(G). If fraction q of WORK(G) is sequential, it must be the case that SPAN(G) ≥ q × WORK(G). Therefore, Speedup(P) = T1/TP must be ≤ WORK(G)/(q × WORK(G)) = 1/q since T1 = WORK(G) for greedy schedulers.
Amdahl’s Law reminds us to watch out for sequential bottlenecks both when designing parallel algorithms and when implementing programs on real machines. As an example, if q = 10%, then Amdahl’s Law reminds us that the best possible speedup must be ≤ 10 (which equals 1/q ), regardless of the number of processors available.
# 1.5 Mini Project
Your main goals for this assignment are as follows:
1.	Modify the ReciprocalArraySum.parArraySum() method to implement the reciprocal-array-sum computation in parallel using the Java Fork Join framework by partitioning the input array in half and computing the reciprocal array sum on the first and second half in parallel, before combining the results. There are TODOs in the source file to guide you, and you are free to refer to the lecture and demonstration videos.
Note that Goal #2 below is a generalization of this goal, so If you are already confident in your understanding of how to partition work among multiple Fork Join tasks (not just two), you are free to skip ahead to Goal #2 below and then implement parArraySum() for Goal #1 by calling the parManyTaskArraySum() method that you will implement for Goal #2 by requesting only two tasks. Otherwise, it is recommended that you work on Goal #1 first, and then proceed to Goal #2.
Note that to complete this and the following goal you will want to explicitly create a ForkJoinPool inside of parArraySum() and parManyTaskArraySum() to run your tasks inside. For example, creating a ForkJoinPool with 2 threads requires the following code:
import java.util.concurrent.ForkJoinPool; ForkJoinPool pool = new ForkJoinPool(2); 
  
1.	Modify the ReciprocalArraySum.parManyTaskArraySum method to implement the reciprocal-array-sum computation in parallel using Java’s Fork Join framework again, but using a given number of tasks (not just two). Note that the getChunkStartInclusive and getChunkEndExclusive utility methods are provided for your convenience to help with calculating the region of the input array a certain task should process.
```
package edu.coursera.parallel;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveAction;

/**
 * Class wrapping methods for implementing reciprocal array sum in parallel.
 */
public final class ReciprocalArraySum {

    /**
     * Default constructor.
     */
    private ReciprocalArraySum() {
    }

    /**
     * Sequentially compute the sum of the reciprocal values for a given array.
     *
     * @param input Input array
     * @return The sum of the reciprocals of the array input
     */
    protected static double seqArraySum(final double[] input) {
        double sum = 0;

        // Compute sum of reciprocals of array elements
        for (int i = 0; i < input.length; i++) {
            sum += 1 / input[i];
        }

        return sum;
    }

    /**
     * Computes the size of each chunk, given the number of chunks to create
     * across a given number of elements.
     *
     * @param nChunks   The number of chunks to create
     * @param nElements The number of elements to chunk across
     * @return The default chunk size
     */
    private static int getChunkSize(final int nChunks, final int nElements) {
        // Integer ceil
        return (nElements + nChunks - 1) / nChunks;
    }

    /**
     * Computes the inclusive element index that the provided chunk starts at,
     * given there are a certain number of chunks.
     *
     * @param chunk     The chunk to compute the start of
     * @param nChunks   The number of chunks created
     * @param nElements The number of elements to chunk across
     * @return The inclusive index that this chunk starts at in the set of
     * nElements
     */
    private static int getChunkStartInclusive(final int chunk,
                                              final int nChunks, final int nElements) {
        final int chunkSize = getChunkSize(nChunks, nElements);
        return chunk * chunkSize;
    }

    /**
     * Computes the exclusive element index that the provided chunk ends at,
     * given there are a certain number of chunks.
     *
     * @param chunk     The chunk to compute the end of
     * @param nChunks   The number of chunks created
     * @param nElements The number of elements to chunk across
     * @return The exclusive end index for this chunk
     */
    private static int getChunkEndExclusive(final int chunk, final int nChunks,
                                            final int nElements) {
        final int chunkSize = getChunkSize(nChunks, nElements);
        final int end = (chunk + 1) * chunkSize;
        if (end > nElements) {
            return nElements;
        } else {
            return end;
        }
    }

    /**
     * This class stub can be filled in to implement the body of each task
     * created to perform reciprocal array sum in parallel.
     */
    private static class ReciprocalArraySumTask extends RecursiveAction {
        /**
         * Starting index for traversal done by this task.
         */
        private final int startIndexInclusive;
        /**
         * Ending index for traversal done by this task.
         */
        private final int endIndexExclusive;
        /**
         * Input array to reciprocal sum.
         */
        private final double[] input;
        /**
         * Intermediate value produced by this task.
         */
        private double value;

        /**
         * Constructor.
         *
         * @param setStartIndexInclusive Set the starting index to begin
         *                               parallel traversal at.
         * @param setEndIndexExclusive   Set ending index for parallel traversal.
         * @param setInput               Input values
         */
        ReciprocalArraySumTask(final int setStartIndexInclusive,
                               final int setEndIndexExclusive, final double[] setInput) {
            this.startIndexInclusive = setStartIndexInclusive;
            this.endIndexExclusive = setEndIndexExclusive;
            this.input = setInput;
        }

        /**
         * Getter for the value produced by this task.
         *
         * @return Value produced by this task
         */
        public double getValue() {
            return value;
        }

        @Override
        protected void compute() {
            int THRESHOLD = 1000;
            if (endIndexExclusive - startIndexInclusive <= THRESHOLD) {
                for (int i = startIndexInclusive; i < endIndexExclusive; i++) {
                    value += 1 / input[i];
                }
            } else {
                int half = (startIndexInclusive + endIndexExclusive) / 2;
                ReciprocalArraySumTask left = new ReciprocalArraySumTask(startIndexInclusive, half, input);
                ReciprocalArraySumTask right = new ReciprocalArraySumTask(half, endIndexExclusive, input);
                left.fork();
                right.compute();
                left.join();
                value += left.getValue() + right.getValue();
            }
        }
    }

    /**
     * TODO: Modify this method to compute the same reciprocal sum as
     * seqArraySum, but use two tasks running in parallel under the Java Fork
     * Join framework. You may assume that the length of the input array is
     * evenly divisible by 2.
     *
     * @param input Input array
     * @return The sum of the reciprocals of the array input
     */
    protected static double parArraySum(final double[] input) {
        assert input.length % 2 == 0;

//        double sum = 0;
//
//        // Compute sum of reciprocals of array elements
//        for (int i = 0; i < input.length; i++) {
//            sum += 1 / input[i];
//        }
//
//        return sum;


        ReciprocalArraySumTask t = new ReciprocalArraySumTask(0, input.length, input);
        ForkJoinPool.commonPool().invoke(t);
        return t.getValue();
    }

    /**
     * TODO: Extend the work you did to implement parArraySum to use a set
     * number of tasks to compute the reciprocal array sum. You may find the
     * above utilities getChunkStartInclusive and getChunkEndExclusive helpful
     * in computing the range of element indices that belong to each chunk.
     *
     * @param input    Input array
     * @param numTasks The number of tasks to create
     * @return The sum of the reciprocals of the array input
     */
    protected static double parManyTaskArraySum(final double[] input,
                                                final int numTasks) {
//        double sum = 0;

        // Compute sum of reciprocals of array elements
//        for (int i = 0; i < input.length; i++) {
//            sum += 1 / input[i];
//        }
//
//        return sum;

        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", String.valueOf(4));
//
//        ReciprocalArraySumTask t = new ReciprocalArraySumTask(0, input.length, input);
//        ForkJoinPool.commonPool().invoke(t);
//        return t.getValue();
//
//        System.out.println("NUM = " + numTasks);
        List<ReciprocalArraySumTask> reciprocalArraySumTasks = new ArrayList<>();
        double sum = 0;

        for (int i = 0; i < numTasks; i++) {
            int start = getChunkStartInclusive(i, numTasks, input.length);
            int end = getChunkEndExclusive(i, numTasks, input.length);
//            System.out.println(i + " : " + start + " : " + end);

            ReciprocalArraySumTask t = new ReciprocalArraySumTask(start, end, input);
//            reciprocalArraySumTasks.add(t);
            ForkJoinPool.commonPool().invoke(t);
            sum += t.getValue();
        }
//        for (ReciprocalArraySumTask task : reciprocalArraySumTasks) {
//            sum += task.getValue();
//        }
        return sum;
    }
}
```
## 2 Functional Parallelism
Advocates of parallel functional programming have argued for decades that functional parallelism can eliminate many hard-to-detect bugs that can occur with imperative parallelism. We will learn about futures, memoization, and streams, as well as data races, a notorious class of bugs that can be avoided with functional parallelism. We will also learn Java APIs for functional parallelism, including the Fork/Join framework and the Stream API’s.
# 2.1 Future Tasks
Future tasks are tasks with return values, and a future object is a “handle” for accessing a task’s return value. There are two key operations that can be performed on a future object, A:
Assignment — A can be assigned a reference to a future object returned by a task of the form, future { ⟨ task-with-return-value ⟩ } (using pseudocode notation). The content of the future object is constrained to be single assignment (similar to a final variable in Java), and cannot be modified after the future task has returned.
Blocking read — the operation, A.get(), waits until the task associated with future object A has completed, and then propagates the task’s return value as the value returned by A.get(). Any statement, S, executed after A.get() can be assured that the task associated with future object A must have completed before S starts execution. These operations are carefully defined to avoid the possibility of a race condition on a task’s return value, which is why futures are well suited for functional parallelism. In fact, one of the earliest use of futures for parallel computing was in an extension to Lisp known as MultiLisp.
 
# 2.2 Creating Future Tasks in Java’s Fork/Join Framework
express future tasks in Java’s Fork/Join (FJ) framework. Some key differences between future tasks and regular tasks in the FJ framework are as follows:
1.	A future task extends the RecursiveTask class in the FJ framework, instead of RecursiveAction as in regular tasks.
2.	The 𝚌𝚘𝚖𝚙𝚞𝚝𝚎() method of a future task must have a non-void return type, whereas it has a void return type for regular tasks.
3.	A method call like 𝚕𝚎𝚏𝚝.𝚓𝚘𝚒𝚗() waits for the task referred to by object 𝚕𝚎𝚏𝚝 in both cases, but also provides the task’s return value in the case of future tasks.
 
# 2.3 Memoization
Memoization is to remember results of function calls f (x) as follows:
1.	Create a data structure that stores the set {(x1, y1 = f (x1)), (x2, y2 = f (x2)), . . .} for each call f (xi) that returns yi.
2.	Perform look ups in that data structure when processing calls of the form f (x’) when x’ equals one of the xi inputs for which f (xi) has already been computed.
Memoization can be especially helpful for algorithms based on dynamic programming. In the lecture, we used Pascal’s triangle as an illustrative example to motivate memoization.
The memoization pattern lends itself easily to parallelization using futures by modifying the memoized data structure to store {(x1, y1 = future(f (x1))), (x2, y2 = future(f (x2))), . . .}. The lookup operation can then be replaced by a get() operation on the future value, if a future has already been created for the result of a given input.
 
# 2.4 Java Streams
Java Stream can provide a functional approach to operating on collections of data. For example, the statement, “students.stream().forEach(s → System.out.println(s));”, is a succinct way of specifying an action to be performed on each element s in the collection, students. An aggregate data query or data transformation can be specified by building a stream pipeline consisting of a source (typically by invoking the .stream() method on a data collection , a sequence of intermediate operations such as map() and filter(), and an optional terminal operation such as forEach() or average(). As an example, the following pipeline can be used to compute the average age of all active students using Java streams:
students.stream() .filter(s -> s.getStatus() == Student.ACTIVE) .map(a -> a.getAge()) .average(); 
   From the viewpoint of this course, an important benefit of using Java streams when possible is that the pipeline can be made to execute in parallel by designating the source to be a parallel stream, i.e., by simply replacing students.stream() in the above code by students.parallelStream() or Stream.of(students).parallel(). This form of functional parallelism is a major convenience for the programmer, since they do not need to worry about explicitly allocating intermediate collections (e.g., a collection of all active students), or about ensuring that parallel accesses to data collections are properly synchronized.
 
# 2.5 Determinism and Data Races
A parallel program is said to be functionally deterministic if it always computes the same answer when given the same input, and structurally deterministic if it always computes the same computation graph, when given the same input. The presence of data races often leads to functional and/or structural nondeterminism because a parallel program with data races may exhibit different behaviors for the same input, depending on the relative scheduling and timing of memory accesses involved in a data race. In general, the absence of data races is not sufficient to guarantee determinism. However, all the parallel constructs introduced in this course (“Parallelism”) were carefully selected to ensure the following Determinism Property:
If a parallel program is written using the constructs introduced in this course and is guaranteed to never exhibit a data race, then it must be both functionally and structurally deterministic.
ß Note that the determinism property states that all data-race-free parallel programs written using the constructs introduced in this course are guaranteed to be deterministic, but it does not imply that a program with a data race must be functionally/structurally non-deterministic. Furthermore, there may be cases of “benign” nondeterminism for programs with data races in which different executions with the same input may generate different outputs, but all the outputs may be acceptable in the context of the application, e.g., different locations for a search pattern in a target string.
 
# 2.6 Mini Project
Your main goals for this assignment are listed below. StudentAnalytics.java also contains helpful TODOs.
Implement StudentAnalytics.averageAgeOfEnrolledStudentsParallelStream to perform the same operations as averageAgeOfEnrolledStudentsImperative but using parallel streams. Implement StudentAnalytics. mostCommonFirstNameOfInactiveStudentsParallelStream to perform the same operations as mostCommonFirstNameOfInactiveStudentsImperative but using parallel streams. Implement StudentAnalytics. countNumberOfFailedStudentsOlderThan20ParallelStream to perform the same operations as countNumberOfFailedStudentsOlderThan20Imperative but using parallel streams. Note that any grade below a 65 is considered a failing grade for the purpose of this method.
```
package edu.coursera.parallel;

import java.util.*;
import java.util.function.Function;
import java.util.stream.Collectors;
import java.util.stream.Stream;

/**
 * A simple wrapper class for various analytics methods.
 */
public final class StudentAnalytics {
    /**
     * Sequentially computes the average age of all actively enrolled students
     * using loops.
     *
     * @param studentArray Student data for the class.
     * @return Average age of enrolled students
     */
    public double averageAgeOfEnrolledStudentsImperative(
            final Student[] studentArray) {
        List<Student> activeStudents = new ArrayList<Student>();

        for (Student s : studentArray) {
            if (s.checkIsCurrent()) {
                activeStudents.add(s);
            }
        }

        double ageSum = 0.0;
        for (Student s : activeStudents) {
            ageSum += s.getAge();
        }

        return ageSum / (double) activeStudents.size();
    }

    /**
     * TODO compute the average age of all actively enrolled students using
     * parallel streams. This should mirror the functionality of
     * averageAgeOfEnrolledStudentsImperative. This method should not use any
     * loops.
     *
     * @param studentArray Student data for the class.
     * @return Average age of enrolled students
     */
    public double averageAgeOfEnrolledStudentsParallelStream(
            final Student[] studentArray) {
        return Stream.of(studentArray)
                .parallel()
                .filter(Student::checkIsCurrent)
                .mapToDouble(Student::getAge)
                .average()
                .getAsDouble();
    }

    /**
     * Sequentially computes the most common first name out of all students that
     * are no longer active in the class using loops.
     *
     * @param studentArray Student data for the class.
     * @return Most common first name of inactive students
     */
    public String mostCommonFirstNameOfInactiveStudentsImperative(
            final Student[] studentArray) {
        List<Student> inactiveStudents = new ArrayList<Student>();

        for (Student s : studentArray) {
            if (!s.checkIsCurrent()) {
                inactiveStudents.add(s);
            }
        }

        Map<String, Integer> nameCounts = new HashMap<String, Integer>();

        for (Student s : inactiveStudents) {
            if (nameCounts.containsKey(s.getFirstName())) {
                nameCounts.put(s.getFirstName(),
                        new Integer(nameCounts.get(s.getFirstName()) + 1));
            } else {
                nameCounts.put(s.getFirstName(), 1);
            }
        }

        String mostCommon = null;
        int mostCommonCount = -1;
        for (Map.Entry<String, Integer> entry : nameCounts.entrySet()) {
            if (mostCommon == null || entry.getValue() > mostCommonCount) {
                mostCommon = entry.getKey();
                mostCommonCount = entry.getValue();
            }
        }

        return mostCommon;
    }

    /**
     * TODO compute the most common first name out of all students that are no
     * longer active in the class using parallel streams. This should mirror the
     * functionality of mostCommonFirstNameOfInactiveStudentsImperative. This
     * method should not use any loops.
     *
     * @param studentArray Student data for the class.
     * @return Most common first name of inactive students
     */
    public String mostCommonFirstNameOfInactiveStudentsParallelStream(
            final Student[] studentArray) {
        return Stream.of(studentArray)
                .filter(s -> !s.checkIsCurrent())
                .map(Student::getFirstName)
                .parallel()
                .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()))
                .entrySet()
                .stream()
                .max(Comparator.comparing(Map.Entry::getValue))
                .get()
                .getKey();
    }

    /**
     * Sequentially computes the number of students who have failed the course
     * who are also older than 20 years old. A failing grade is anything below a
     * 65. A student has only failed the course if they have a failing grade and
     * they are not currently active.
     *
     * @param studentArray Student data for the class.
     * @return Number of failed grades from students older than 20 years old.
     */
    public int countNumberOfFailedStudentsOlderThan20Imperative(
            final Student[] studentArray) {
        int count = 0;
        for (Student s : studentArray) {
            if (!s.checkIsCurrent() && s.getAge() > 20 && s.getGrade() < 65) {
                count++;
            }
        }
        return count;
    }

    /**
     * TODO compute the number of students who have failed the course who are
     * also older than 20 years old. A failing grade is anything below a 65. A
     * student has only failed the course if they have a failing grade and they
     * are not currently active. This should mirror the functionality of
     * countNumberOfFailedStudentsOlderThan20Imperative. This method should not
     * use any loops.
     *
     * @param studentArray Student data for the class.
     * @return Number of failed grades from students older than 20 years old.
     */
    public int countNumberOfFailedStudentsOlderThan20ParallelStream(
            final Student[] studentArray) {
        return Stream.of(studentArray)
                .parallel()
                .filter(s -> !s.checkIsCurrent() && s.getAge() > 20 && s.getGrade() < 65)
                .mapToInt(s -> 1)
                .sum();
    }
}
```
## 3 Loop Parallelism
# 3.1 Parallel Loops
The most general way is to think of each iteration of a parallel loop as an async task, with a finish construct encompassing all iterations. This approach can support general cases such as parallelization of the following pointer-chasing while loop (in pseudocode):
finish {
for (p = head; p != null ; p = p.next) async compute(p);
}
   However, further efficiencies can be gained by paying attention to counted-for loops for which the number of iterations is known on entry to the loop (before the loop executes its first iteration). We then learned the forall notation for expressing parallel counted-for loops, such as in the following vector addition statement (in pseudocode):
forall (i : [0:n-1]) a[i] = b[i] + c[i]
   We also discussed the fact that Java streams can be an elegant way of specifying parallel loop computations that produce a single output array, e.g., by rewriting the vector addition statement as follows:
a = IntStream.rangeClosed(0, N-1).parallel().toArray(i -> b[i] + c[i]);
   In summary, streams are a convenient notation for parallel loops with at most one output array, but the forall notation is more convenient for loops that create/update multiple output arrays, as is the case in many scientific computations. For generality, we will use the forall notation for parallel loops in the remainder of this module.
 
# 3.2 Parallel Matrix Multiplication
c [ i ][ j ] = ∑n−1k=0 a[ i ][ k ] ∗ b[ k ][ j ]
for(i : [0:n-1]) {
  for(j : [0:n-1]) { c[i][j] = 0;
    for(k : [0:n-1]) {
      c[i][j] = c[i][j] + a[i][k]*b[k][j]
    }
  }
}
# 3.3 Barriers in Parallel Loops
forall (i : [0:n-1]) { myId = lookup(i); // convert int to a string print HELLO, myId; print BYE, myId; } 
the HELLO’s and BYE’s from different forall iterations may be interleaved in the printed output, e.g., some HELLO’s may follow some BYE’s. Then, we showed how inserting a barrier between the two print statements could ensure that all HELLO’s would be printed before any BYE’s.
Thus, barriers extend a parallel loop by dividing its execution into a sequence of phases. While it may be possible to write a separate forall loop for each phase, it is both more convenient and more efficient to instead insert barriers in a single forall loop, e.g., we would need to create an intermediate data structure to communicate the myId values from one forall to another forall if we split the above forall into two (using the notation next) loops. Barriers are a fundamental construct for parallel loops that are used in a majority of real-world parallel applications.
 
# 3.4 One-Dimensional Iterative Averaging
The Jacobi method for solving such equations typically utilizes two arrays, oldX[] and newX[]. A naive approach to parallelizing this method would result in the following pseudocode:
for (iter: [0:nsteps-1]) { forall (i: [1:n-1]) { newX[i] = (oldX[i-1] + oldX[i+1]) / 2; } swap pointers newX and oldX; } 
Though easy to understand, this approach creates nsteps × (n − 1) tasks, which is too many. Barriers can help reduce the number of tasks created as follows:
forall ( i: [1:n-1]) { for (iter: [0:nsteps-1]) { newX[i] = (oldX[i-1] + oldX[i+1]) / 2; NEXT; // Barrier swap pointers newX and oldX; } } 
In this case, only (n − 1) tasks are created, and there are nsteps barrier (next) operations, each of which involves all (n − 1) tasks. This is a significant improvement since creating tasks is usually more expensive than performing barrier operations.
 
# 3.5 Iteration Grouping: Chunking of Parallel Loops
we revisited the vector addition example:
forall (i : [0:n-1]) a[i] = b[i] + c[i]
We observed that this approach creates n tasks, one per forall iteration, which is wasteful when (as is common in practice) n is much larger than the number of available processor cores.
To address this problem, we learned a common tactic used in practice that is referred to as loop chunking or iteration grouping, and focuses on reducing the number of tasks created to be closer to the number of processor cores, so as to reduce the overhead of parallel execution:
With iteration grouping/chunking, the parallel vector addition example above can be rewritten as follows:
forall (g:[0:ng-1])
  for (i : mygroup(g, ng, [0:n-1])) a[i] = b[i] + c[i]
Note that we have reduced the degree of parallelism from n to the number of groups, ng, which now equals the number of iterations/tasks in the forall construct.
There are two well known approaches for iteration grouping: block and cyclic. The former approach (block) maps consecutive iterations to the same group, whereas the latter approach (cyclic) maps iterations in the same congruence class (mod ng) to the same group. With these concepts, you should now have a better understanding of how to execute forall loops in practice with lower overhead.
For convenience, the PCDP library provides helper methods, forallChunked() and forall2dChunked(), that automatically create one-dimensional or two-dimensional parallel loops, and also perform a block-style iteration grouping/chunking on those parallel loops. An example of using the forall2dChunked() API for a two-dimensional parallel loop (as in the matrix multiply example) can be seen in the following Java code sketch:
forall2dChunked(0, N - 1, 0, N - 1, (i, j) -> {
    . . . // Statements for parallel iteration (i,j)
 });
 
# 3.6 Mini Project
Modify the MatrixMultiply.parMatrixMultiply method to implement matrix multiply in parallel using PCDP’s forall or forallChunked methods. This will closely follow the demo by Prof. Sarkar in the first demo video of Week 3. There is one TODO in MatrixMultiply.parMatrixMultiply to help indicate the changes to be made. A parallel implementation of MatrixMultiply.parMatrixMultiply should result in near-linear speedup (i.e. the speedup achieved should be close to the number of cores in your machine).
package edu.coursera.parallel;
```
import static edu.rice.pcdp.PCDP.forall2dChunked;
import static edu.rice.pcdp.PCDP.forasyncChunked;
import static edu.rice.pcdp.PCDP.forseq2d;

/**
 * Wrapper class for implementing matrix multiply efficiently in parallel.
 */
public final class MatrixMultiply {
    /**
     * Default constructor.
     */
    private MatrixMultiply() {
    }

    /**
     * Perform a two-dimensional matrix multiply (A x B = C) sequentially.
     *
     * @param A An input matrix with dimensions NxN
     * @param B An input matrix with dimensions NxN
     * @param C The output matrix
     * @param N Size of each dimension of the input matrices
     */
    public static void seqMatrixMultiply(final double[][] A, final double[][] B,
                                         final double[][] C, final int N) {
        forseq2d(0, N - 1, 0, N - 1, (i, j) -> {
            C[i][j] = 0.0;
            for (int k = 0; k < N; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        });
    }

    /**
     * Perform a two-dimensional matrix multiply (A x B = C) in parallel.
     *
     * @param A An input matrix with dimensions NxN
     * @param B An input matrix with dimensions NxN
     * @param C The output matrix
     * @param N Size of each dimension of the input matrices
     */
    public static void parMatrixMultiply(final double[][] A, final double[][] B,
                                         final double[][] C, final int N) {
        /*
         * TODO Parallelize this outermost two-dimension sequential loop to
         * achieve performance improvement.
         */
        forall2dChunked(0, N - 1, 0, N - 1, N, (i, j) -> {
            C[i][j] = 0.0;
            for (int k = 0; k < N; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        });
    }
}
```

## 4 Dataflow Synchronization and Pipelining
# 4.1 Split-phase Barriers with Java Phasers
We learned about Java’s Phaser class, and that the operation ph.arriveAndAwaitAdvance(), can be used to implement a barrier through phaser object ph. We also observed that there are two possible positions for inserting a barrier between the two print statements above — before or after the call to lookup(i). However, upon closer examination, we can see that the call to lookup(i) is local to iteration i and that there is no specific need to either complete it before the barrier or to complete it after the barrier. In fact, the call to lookup(i) can be performed in parallel with the barrier. To facilitate this split-phase barrier (also known as a fuzzy barrier) we use two separate APIs from Java Phaser class — ph.arrive() and ph.awaitAdvance(). Together these two APIs form a barrier, but we now have the freedom to insert a computation such as lookup(i) between the two calls as follows:
// initialize phaser ph for use by n tasks ("parties") Phaser ph = new Phaser(n); // Create forall loop with n iterations that operate on ph forall (i : [0:n-1]) { print HELLO, i; int phase = ph.arrive(); myId = lookup(i); // convert int to a string ph.awaitAdvance(phase); print BYE, myId; } 
Doing so enables the barrier processing to occur in parallel with the call to lookup(i), which was our desired outcome.
 
# 4.2 Point-to-Point Synchronization with Phasers
we looked at a parallel program example in which the span (critical path length) would be 6 units of time if we used a barrier, but is reduced to 5 units of time if we use individual phasers as shown in the following table:
 
Each column in the table represents execution of a separate task, and the calls to arrive() and awaitAdvance(0) represent synchronization across different tasks via phaser objects, ph0, ph1, and ph2, each of which is initialized with a party count of 1 (only one signalling task). (The parameter 0 in awaitAdvance(0) represents a transition from phase 0 to phase 1.)
 
# 4.3 One-Dimensional Iterative Averaging with Phasers
a full barrier is not necessary since forall iteration i only needs to wait for iterations i − 1 and i + 1 to complete their current phase before iteration i can move to its next phase. This idea can be captured by phasers, if we allocate an array of phasers as follows:
// Allocate array of phasers
Phaser[] ph = new Phaser[n+2]; //array of phasers
for (int i = 0; i < ph.length; i++) ph[i] = new Phaser(1);

// Main computation
forall ( i: [1:n-1]) {
  for (iter: [0:nsteps-1]) {
    newX[i] = (oldX[i-1] + oldX[i+1]) / 2;
    ph[i].arrive();

    if (index > 1) ph[i-1].awaitAdvance(iter);
    if (index < n-1) ph[i + 1].awaitAdvance(iter);
    swap pointers newX and oldX;
  }
}
As we learned earlier, grouping/chunking of parallel iterations in a forall can be an important consideration for performance (due to reduced overhead). The idea of grouping of parallel iterations can be extended to forall loops with phasers as follows:

// Allocate array of phasers proportional to number of chunked tasks
Phaser[] ph = new Phaser[tasks+2]; //array of phasers
for (int i = 0; i < ph.length; i++) ph[i] = new Phaser(1);

// Main computation
forall ( i: [0:tasks-1]) {
  for (iter: [0:nsteps-1]) {
    // Compute leftmost boundary element for group
    int left = i * (n / tasks) + 1;
    myNew[left] = (myVal[left - 1] + myVal[left + 1]) / 2.0;

    // Compute rightmost boundary element for group
    int right = (i + 1) * (n / tasks);
    myNew[right] = (myVal[right - 1] + myVal[right + 1]) / 2.0;

    // Signal arrival on phaser ph AND LEFT AND RIGHT ELEMENTS ARE AV
    int	index = i + 1;
    ph[index].arrive();

    // Compute interior elements in parallel with barrier
    for (int j = left + 1; j <= right - 1; j++)
      myNew[j] = (myVal[j - 1] + myVal[j + 1]) / 2.0;
    // Wait for previous phase to complete before advancing
    if (index > 1) ph[index - 1].awaitAdvance(iter);
    if (index < tasks) ph[index + 1].awaitAdvance(iter);
    swap pointers newX and oldX;
  }
}

 
# 4.4 Pipeline Parallelism
point-to-point synchronization can be used to build a one-dimensional pipeline with p tasks (stages), T_0 , . . . , Tp−1. For example, three important stages in a medical imaging pipeline are denoising, registration, and segmentation.
We performed a simplified analysis of the WORK and SPAN for pipeline parallelism as follows. Let n be the number of input items and p the number of stages in the pipeline, WORK = n × p is the total work that must be done for all data items, and CPL = n + p − 1 is the span or critical path length for the pipeline. Thus, the ideal parallelism is PAR = WORK /CPL = np / (n + p − 1). This formula can be validated by considering a few boundary cases. When p = 1, the ideal parallelism degenerates to PAR = 1, which confirms that the computation is sequential when only one stage is available. Likewise, when n = 1, the ideal parallelism again degenerates to PAR = 1, which confirms that the computation is sequential when only one data item is available. When n is much larger than p (n » p), then the ideal parallelism approaches PAR = p in the limit, which is the best possible case.
The synchronization required for pipeline parallelism can be implemented using phasers by allocating an array of phasers, such that phaser {\tt ph[i]}ph[i] is “signalled” in iteration i by a call to ph[i].arrive() as follows:
// Code for pipeline stage i
while ( there is an input to be processed ) {
  // wait for previous stage, if any
  if (i > 0) ph[i - 1].awaitAdvance();

  process input;

  // signal next stage
  ph[i].arrive();
}
 
# 4.5 Data Flow Parallelism
Thus far, we have studied computation graphs as structures that are derived from parallel programs. In this lecture, we studied a dual approach advocated in the data flow parallelism model, which is to specify parallel programs as computation graphs. The simple data flow graph studied in the lecture consisted of five nodes and four edges: A → C, A → D, B → D, B → E. While futures can be used to generate such a computation graph, e.g., by including calls to A.get() and B.get() in task D, the computation graph edges are implicit in the get() calls when using futures. Instead, we introduced the asyncAwait notation to specify a task along with an explicit set of preconditions (events that the task must wait for before it can start execution). With this approach, the program can be generated directly from the computation graph as follows:
async( () -> {/* Task A */; A.put(); } ); // Complete task and trigger event A async( () -> {/* Task B */; B.put(); } ); // Complete task and trigger event B asyncAwait(A, () -> {/* Task C */} ); // Only execute task after event A is triggered asyncAwait(A, B () -> {/* Task D */} ); // Only execute task after events A, B are triggered asyncAwait(B, () -> {/* Task E */} ); // Only execute task after event B is triggered 
Interestingly, the order of the above statements is not significant. Just as a graph can be defined by enumerating its edges in any order, the above data flow program can be rewritten as follows, without changing its meaning:
asyncAwait(A, () -> {/* Task C */} ); // Only execute task after event A is triggered asyncAwait(A, B () -> {/* Task D */} ); // Only execute task after events A, B are triggered asyncAwait(B, () -> {/* Task E */} ); // Only execute task after event B is triggered async( () -> {/* Task A */; A.put(); } ); // Complete task and trigger event A async( () -> {/* Task B */; B.put(); } ); // Complete task and trigger event B 
Finally, we observed that the power and elegance of data flow parallel programming is accompanied by the possibility of a lack of progress that can be viewed as a form of “deadlock” if the program omits a put() call for signalling an event.
 
# 4.6 Mini Project
Your main goals for this assignment are listed below. OneDimAveragingPhaser.java also contains helpful TODOs.
Based on the provided reference sequential version in OneDimAveragingPhaser.runSequential and the reference parallel version in OneDimAveragingPhaser.runParallelBarrier, implement a parallel version of the one-dimensional iterative averaging algorithm that uses phasers to maximize the overlap between barrier completion and useful work being completed.
package edu.coursera.parallel;

import java.util.concurrent.Phaser;

/**
 * Wrapper class for implementing one-dimensional iterative averaging using
 * phasers.
 */
public final class OneDimAveragingPhaser {
    /**
     * Default constructor.
     */
    private OneDimAveragingPhaser() {
    }

    /**
     * Sequential implementation of one-dimensional iterative averaging.
     *
     * @param iterations The number of iterations to run
     * @param myNew A double array that starts as the output array
     * @param myVal A double array that contains the initial input to the
     *        iterative averaging problem
     * @param n The size of this problem
     */
    public static void runSequential(final int iterations, final double[] myNew,
            final double[] myVal, final int n) {
        double[] next = myNew;
        double[] curr = myVal;

        for (int iter = 0; iter < iterations; iter++) {
            for (int j = 1; j <= n; j++) {
                next[j] = (curr[j - 1] + curr[j + 1]) / 2.0;
            }
            double[] tmp = curr;
            curr = next;
            next = tmp;
        }
    }

    /**
     * An example parallel implementation of one-dimensional iterative averaging
     * that uses phasers as a simple barrier (arriveAndAwaitAdvance).
     *
     * @param iterations The number of iterations to run
     * @param myNew A double array that starts as the output array
     * @param myVal A double array that contains the initial input to the
     *        iterative averaging problem
     * @param n The size of this problem
     * @param tasks The number of threads/tasks to use to compute the solution
     */
    public static void runParallelBarrier(final int iterations,
            final double[] myNew, final double[] myVal, final int n,
            final int tasks) {
        Phaser ph = new Phaser(0);
        ph.bulkRegister(tasks);

        Thread[] threads = new Thread[tasks];

        for (int ii = 0; ii < tasks; ii++) {
            final int i = ii;

            threads[ii] = new Thread(() -> {
                double[] threadPrivateMyVal = myVal;
                double[] threadPrivateMyNew = myNew;

                for (int iter = 0; iter < iterations; iter++) {
                    final int left = i * (n / tasks) + 1;
                    final int right = (i + 1) * (n / tasks);

                    for (int j = left; j <= right; j++) {
                        threadPrivateMyNew[j] = (threadPrivateMyVal[j - 1]
                            + threadPrivateMyVal[j + 1]) / 2.0;
                    }
                    ph.arriveAndAwaitAdvance();

                    double[] temp = threadPrivateMyNew;
                    threadPrivateMyNew = threadPrivateMyVal;
                    threadPrivateMyVal = temp;
                }
            });
            threads[ii].start();
        }

        for (int ii = 0; ii < tasks; ii++) {
            try {
                threads[ii].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * A parallel implementation of one-dimensional iterative averaging that
     * uses the Phaser.arrive and Phaser.awaitAdvance APIs to overlap
     * computation with barrier completion.
     *
     * TODO Complete this method based on the provided runSequential and
     * runParallelBarrier methods.
     *
     * @param iterations The number of iterations to run
     * @param myNew A double array that starts as the output array
     * @param myVal A double array that contains the initial input to the
     *              iterative averaging problem
     * @param n The size of this problem
     * @param tasks The number of threads/tasks to use to compute the solution
     */
    public static void runParallelFuzzyBarrier(final int iterations,
            final double[] myNew, final double[] myVal, final int n,
            final int tasks) {
        Phaser ph = new Phaser(0);
        ph.bulkRegister(tasks);

        Thread[] threads = new Thread[tasks];

        for (int ii = 0; ii < tasks; ii++) {
            final int i = ii;

            threads[ii] = new Thread(() -> {
                double[] threadPrivateMyVal = myVal;
                double[] threadPrivateMyNew = myNew;

                for (int iter = 0; iter < iterations; iter++) {
                    final int left = i * (n / tasks) + 1;
                    threadPrivateMyNew[left] = (threadPrivateMyVal[left - 1] + threadPrivateMyVal[left + 1]) / 2.0;

                    final int right = (i + 1) * (n / tasks);
                    threadPrivateMyNew[right] = (threadPrivateMyVal[right - 1] + threadPrivateMyVal[right + 1]) / 2.0;

                    int currentPhase = ph.arrive();
                    for (int j = left + 1; j <= right - 1; j++) {
                        threadPrivateMyNew[j] = (threadPrivateMyVal[j - 1] + threadPrivateMyVal[j + 1]) / 2.0;
                    }
                    ph.awaitAdvance(currentPhase);

                    double[] temp = threadPrivateMyNew;
                    threadPrivateMyNew = threadPrivateMyVal;
                    threadPrivateMyVal = temp;
                }
            });
            threads[ii].start();
        }

        for (int ii = 0; ii < tasks; ii++) {
            try {
                threads[ii].join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}


