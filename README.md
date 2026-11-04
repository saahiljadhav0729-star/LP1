Practical 1: Two-Pass Assembler
Filename: TwoPassAssembler.java

This program performs Pass 1 to generate an intermediate file and symbol table, then Pass 2 to generate machine code.
import java.io.*;
import java.util.*;

public class TwoPassAssembler {
    static Map<String, Integer> symbols = new LinkedHashMap<>();
    static Map<String, String> opcodes = new HashMap<>();

    static boolean isOperation(String s) {
        return opcodes.containsKey(s.toUpperCase())
                || s.equalsIgnoreCase("DC")
                || s.equalsIgnoreCase("DS");
    }

    public static void main(String[] args) throws Exception {
        opcodes.put("MOV", "01");
        opcodes.put("ADD", "02");
        opcodes.put("SUB", "03");
        opcodes.put("JMP", "04");

        Scanner sc = new Scanner(System.in);
        System.out.print("Enter number of source statements: ");
        int n = sc.nextInt();
        sc.nextLine();

        List<String> source = new ArrayList<>();
        System.out.println("Enter statements:");
        for (int i = 0; i < n; i++)
            source.add(sc.nextLine().trim());

        // PASS 1
        int lc = 0;

        try (PrintWriter out = new PrintWriter("intermediate.txt")) {
            for (String line : source) {
                if (line.isEmpty()) continue;

                String[] p = line.split("\\s+");
                if (p[0].equalsIgnoreCase("START")) {
                    lc = Integer.parseInt(p[1]);
                    continue;
                }
                if (p[0].equalsIgnoreCase("END")) break;

                String label = "";
                String op;
                String operand = "";

                if (isOperation(p[0])) {
                    op = p[0].toUpperCase();
                    if (p.length > 1) operand = p[1];
                } else {
                    label = p[0];
                    op = p[1].toUpperCase();
                    if (p.length > 2) operand = p[2];
                }

                if (!label.isEmpty())
                    symbols.put(label, lc);

                out.println(lc + "|" + op + "|" + operand);

                if (op.equals("DS"))
                    lc += Integer.parseInt(operand);
                else
                    lc++;
            }
        }

        try (PrintWriter out = new PrintWriter("symbol_table.txt")) {
            for (String s : symbols.keySet())
                out.println(s + "|" + symbols.get(s));
        }

        System.out.println("PASS 1 completed.");
        System.out.println("Intermediate code and symbol table created.");

        // PASS 2
        try (BufferedReader br = new BufferedReader(
                    new FileReader("intermediate.txt"));
             PrintWriter out = new PrintWriter("machine_code.txt")) {

            String line;
            while ((line = br.readLine()) != null) {
                String[] p = line.split("\\|", -1);
                int address = Integer.parseInt(p[0]);
                String op = p[1];
                String operand = p[2];

                if (op.equals("DS")) continue;

                if (op.equals("DC")) {
                    out.println(address + "  " + operand);
                    continue;
                }

                int value = 0;
                if (!operand.isEmpty()) {
                    if (symbols.containsKey(operand))
                        value = symbols.get(operand);
                    else
                        value = Integer.parseInt(operand);
                }

                out.println(address + "  " + opcodes.get(op)
                        + "  " + value);
            }
        }

        System.out.println("PASS 2 completed.");
        System.out.println("Machine code saved in machine_code.txt");
        System.out.println("Symbol Table:");
        for (String s : symbols.keySet())
            System.out.println(s + " : " + symbols.get(s));
    }
}


Instructions / sample input: When prompted, enter the following 9 statements, one per line:
START 100
LOOP MOV A
ADD B
SUB C
JMP LOOP
A DC 5
B DC 10
C DC 15
END


The program creates intermediate.txt, symbol_table.txt, and machine_code.txt in the same directory.

Practical 2: Two-Pass Macro Processor
Filename: TwoPassMacroProcessor.java

Pass 1 stores macro definitions in the Macro Name Table (MNT) and Macro Definition Table (MDT). Pass 2 expands macro calls.
import java.io.*;
import java.util.*;

public class TwoPassMacroProcessor {
    public static void main(String[] args) throws Exception {
        List<String> lines = new ArrayList<>();

        try (BufferedReader br = new BufferedReader(
                new FileReader("input.asm"))) {
            String line;
            while ((line = br.readLine()) != null)
                lines.add(line.trim());
        }

        Map<String, String> parameters = new LinkedHashMap<>();
        Map<String, List<String>> macros = new LinkedHashMap<>();
        List<String> intermediate = new ArrayList<>();

        // PASS 1: Read macro definitions
        for (int i = 0; i < lines.size(); i++) {
            if (lines.get(i).equalsIgnoreCase("MACRO")) {
                String header = lines.get(++i);
                String[] h = header.split("\\s+");
                String name = h[0];
                String parameter = h.length > 1 ? h[1] : "";

                List<String> body = new ArrayList<>();

                while (++i < lines.size()
                        && !lines.get(i).equalsIgnoreCase("MEND")) {
                    body.add(lines.get(i));
                }

                macros.put(name, body);
                parameters.put(name, parameter);
            } else {
                intermediate.add(lines.get(i));
            }
        }

        try (PrintWriter mnt = new PrintWriter("MNT.txt");
             PrintWriter mdt = new PrintWriter("MDT.txt")) {
            for (String name : macros.keySet()) {
                mnt.println(name + " " + parameters.get(name));
                for (String instruction : macros.get(name))
                    mdt.println(instruction);
                mdt.println("MEND");
            }
        }

        try (PrintWriter out = new PrintWriter("intermediate.asm")) {
            for (String line : intermediate)
                out.println(line);
        }

        // PASS 2: Expand macro calls
        try (PrintWriter out = new PrintWriter("output.asm")) {
            for (String line : intermediate) {
                String[] parts = line.split("\\s+");
                String name = parts[0];

                if (macros.containsKey(name)) {
                    String argument = parts.length > 1 ? parts[1] : "";
                    String parameter = parameters.get(name);

                    for (String instruction : macros.get(name))
                        out.println(instruction.replace(parameter, argument));
                } else {
                    out.println(line);
                }
            }
        }

        System.out.println("PASS 1 completed: MNT, MDT and intermediate file created.");
        System.out.println("PASS 2 completed: Expanded code saved in output.asm");
    }
}


Required input file: Create a separate file named input.asm in the same folder.
MACRO
INCR &A
LOAD &A
ADD ONE
STORE &A
MEND
START
INCR COUNT
END


Important: Run this program from the folder containing input.asm. It generates MNT.txt, MDT.txt, intermediate.asm, and output.asm.

Practical 3: FCFS CPU Scheduling
Filename: FCFS.java

FCFS (First Come First Serve) executes processes in order of arrival.
import java.util.*;

public class FCFS {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] pid = new int[n], at = new int[n], bt = new int[n];
        int[] wt = new int[n], tat = new int[n];

        for (int i = 0; i < n; i++) {
            pid[i] = i + 1;
            System.out.print("Enter arrival and burst time for P" + (i + 1) + ": ");
            at[i] = sc.nextInt();
            bt[i] = sc.nextInt();
        }

        // Sort processes by arrival time
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (at[i] > at[j]) {
                    int t = at[i]; at[i] = at[j]; at[j] = t;
                    t = bt[i]; bt[i] = bt[j]; bt[j] = t;
                    t = pid[i]; pid[i] = pid[j]; pid[j] = t;
                }
            }
        }

        int time = 0;
        double totalWT = 0, totalTAT = 0;

        System.out.println("\nGantt Chart:");
        for (int i = 0; i < n; i++) {
            if (time < at[i]) time = at[i];

            wt[i] = time - at[i];
            System.out.print("| P" + pid[i] + " ");
            time += bt[i];
            tat[i] = time - at[i];

            totalWT += wt[i];
            totalTAT += tat[i];
        }
        System.out.println("|");

        System.out.println("\nPID\tAT\tBT\tWT\tTAT");
        for (int i = 0; i < n; i++)
            System.out.println("P" + pid[i] + "\t" + at[i] + "\t"
                    + bt[i] + "\t" + wt[i] + "\t" + tat[i]);

        System.out.println("Average Waiting Time: " + totalWT / n);
        System.out.println("Average Turnaround Time: " + totalTAT / n);
    }
}


Instructions: No extra input file is needed. Enter the number of processes, followed by each process's arrival time and burst time.

Practical 4: SJF Preemptive CPU Scheduling
Filename: SJFPreemptive.java

SJF Preemptive is also called Shortest Remaining Time First (SRTF). The process with the shortest remaining burst time executes.
import java.util.*;

public class SJFPreemptive {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] at = new int[n], bt = new int[n], rt = new int[n];
        int[] wt = new int[n], tat = new int[n], ct = new int[n];

        for (int i = 0; i < n; i++) {
            System.out.print("Enter AT and BT for P" + (i + 1) + ": ");
            at[i] = sc.nextInt();
            bt[i] = sc.nextInt();
            rt[i] = bt[i];
        }

        int time = 0, complete = 0;
        double totalWT = 0, totalTAT = 0;

        while (complete < n) {
            int shortest = -1, min = Integer.MAX_VALUE;

            for (int i = 0; i < n; i++) {
                if (at[i] <= time && rt[i] > 0 && rt[i] < min) {
                    min = rt[i];
                    shortest = i;
                }
            }

            if (shortest == -1) {
                time++;
                continue;
            }

            rt[shortest]--;
            time++;

            if (rt[shortest] == 0) {
                complete++;
                ct[shortest] = time;
                tat[shortest] = ct[shortest] - at[shortest];
                wt[shortest] = tat[shortest] - bt[shortest];

                totalWT += wt[shortest];
                totalTAT += tat[shortest];
            }
        }

        System.out.println("\nPID\tAT\tBT\tWT\tTAT");
        for (int i = 0; i < n; i++)
            System.out.println("P" + (i + 1) + "\t" + at[i] + "\t"
                    + bt[i] + "\t" + wt[i] + "\t" + tat[i]);

        System.out.println("Average Waiting Time: " + totalWT / n);
        System.out.println("Average Turnaround Time: " + totalTAT / n);
    }
}


Instructions: Enter arrival time and burst time for each process. No separate file is required.

Practical 5: Priority Scheduling – Non-Preemptive
Filename: PriorityNonPreemptive.java

The process with the highest priority is selected first. In this code, a smaller priority number means higher priority.
import java.util.*;

public class PriorityNonPreemptive {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] at = new int[n], bt = new int[n], pr = new int[n];
        int[] wt = new int[n], tat = new int[n];
        boolean[] done = new boolean[n];

        for (int i = 0; i < n; i++) {
            System.out.print("Enter AT, BT and priority for P" + (i + 1) + ": ");
            at[i] = sc.nextInt();
            bt[i] = sc.nextInt();
            pr[i] = sc.nextInt();
        }

        int time = 0, completed = 0;
        double totalWT = 0, totalTAT = 0;

        while (completed < n) {
            int selected = -1;

            for (int i = 0; i < n; i++) {
                if (!done[i] && at[i] <= time) {
                    if (selected == -1 || pr[i] < pr[selected])
                        selected = i;
                }
            }

            if (selected == -1) {
                time++;
                continue;
            }

            wt[selected] = time - at[selected];
            time += bt[selected];
            tat[selected] = time - at[selected];

            done[selected] = true;
            completed++;

            totalWT += wt[selected];
            totalTAT += tat[selected];
        }

        System.out.println("\nPID\tAT\tBT\tPriority\tWT\tTAT");
        for (int i = 0; i < n; i++)
            System.out.println("P" + (i + 1) + "\t" + at[i] + "\t"
                    + bt[i] + "\t" + pr[i] + "\t\t"
                    + wt[i] + "\t" + tat[i]);

        System.out.println("Average Waiting Time: " + totalWT / n);
        System.out.println("Average Turnaround Time: " + totalTAT / n);
    }
}


Instructions: Enter arrival time, burst time, and priority for each process. If your college manual uses a larger number to mean higher priority, reverse the priority comparison.

Practical 6: Round Robin CPU Scheduling
Filename: RoundRobin.java

Each process gets a fixed time quantum. Processes are executed in cyclic order.
import java.util.*;

public class RoundRobin {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] at = new int[n], bt = new int[n], rt = new int[n];
        int[] ct = new int[n], wt = new int[n], tat = new int[n];

        for (int i = 0; i < n; i++) {
            System.out.print("Enter AT and BT for P" + (i + 1) + ": ");
            at[i] = sc.nextInt();
            bt[i] = sc.nextInt();
            rt[i] = bt[i];
        }

        System.out.print("Enter time quantum: ");
        int quantum = sc.nextInt();

        if (quantum <= 0) {
            System.out.println("Time quantum must be positive.");
            return;
        }

        Queue<Integer> q = new LinkedList<>();
        boolean[] added = new boolean[n];
        int time = 0, completed = 0;

        while (completed < n) {
            for (int i = 0; i < n; i++) {
                if (!added[i] && at[i] <= time) {
                    q.add(i);
                    added[i] = true;
                }
            }

            if (q.isEmpty()) {
                time++;
                continue;
            }

            int p = q.poll();
            int run = Math.min(quantum, rt[p]);

            time += run;
            rt[p] -= run;

            for (int i = 0; i < n; i++) {
                if (!added[i] && at[i] <= time) {
                    q.add(i);
                    added[i] = true;
                }
            }

            if (rt[p] > 0) {
                q.add(p);
            } else {
                ct[p] = time;
                tat[p] = ct[p] - at[p];
                wt[p] = tat[p] - bt[p];
                completed++;
            }
        }

        double totalWT = 0, totalTAT = 0;

        System.out.println("\nPID\tAT\tBT\tWT\tTAT");
        for (int i = 0; i < n; i++) {
            System.out.println("P" + (i + 1) + "\t" + at[i] + "\t"
                    + bt[i] + "\t" + wt[i] + "\t" + tat[i]);
            totalWT += wt[i];
            totalTAT += tat[i];
        }

        System.out.println("Average Waiting Time: " + totalWT / n);
        System.out.println("Average Turnaround Time: " + totalTAT / n);
    }
}


Instructions: Enter arrival time, burst time, and then the time quantum. No separate file is required.

Practical 7: First Fit Memory Allocation
Filename: FirstFit.java

First Fit allocates a process to the first memory block that is large enough.
import java.util.*;

public class FirstFit {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of memory blocks: ");
        int m = sc.nextInt();

        int[] block = new int[m];

        System.out.println("Enter block sizes:");
        for (int i = 0; i < m; i++)
            block[i] = sc.nextInt();

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] process = new int[n];
        int[] allocation = new int[n];

        System.out.println("Enter process sizes:");
        for (int i = 0; i < n; i++) {
            process[i] = sc.nextInt();
            allocation[i] = -1;
        }

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (block[j] >= process[i]) {
                    allocation[i] = j;
                    block[j] -= process[i];
                    break;
                }
            }
        }

        System.out.println("\nProcess\tSize\tBlock");
        for (int i = 0; i < n; i++) {
            System.out.print("P" + (i + 1) + "\t" + process[i] + "\t");
            if (allocation[i] != -1)
                System.out.println(allocation[i] + 1);
            else
                System.out.println("Not Allocated");
        }
    }
}


Instructions: Enter the number of blocks, block sizes, number of processes, and process sizes. No extra file is needed.

Practical 8: Best Fit Memory Allocation
Filename: BestFit.java

Best Fit allocates a process to the smallest available block that can accommodate it.
import java.util.*;

public class BestFit {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of blocks: ");
        int m = sc.nextInt();

        int[] block = new int[m];
        System.out.println("Enter block sizes:");
        for (int i = 0; i < m; i++)
            block[i] = sc.nextInt();

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] process = new int[n], allocation = new int[n];

        System.out.println("Enter process sizes:");
        for (int i = 0; i < n; i++) {
            process[i] = sc.nextInt();
            allocation[i] = -1;
        }

        for (int i = 0; i < n; i++) {
            int best = -1;

            for (int j = 0; j < m; j++) {
                if (block[j] >= process[i]) {
                    if (best == -1 || block[j] < block[best])
                        best = j;
                }
            }

            if (best != -1) {
                allocation[i] = best;
                block[best] -= process[i];
            }
        }

        System.out.println("\nProcess\tSize\tBlock");
        for (int i = 0; i < n; i++) {
            System.out.print("P" + (i + 1) + "\t" + process[i] + "\t");
            if (allocation[i] != -1)
                System.out.println(allocation[i] + 1);
            else
                System.out.println("Not Allocated");
        }
    }
}


Instructions: Enter block sizes and process sizes when prompted. No extra file is needed.

Practical 9: Next Fit Memory Allocation
Filename: NextFit.java

Next Fit continues searching from the position where the previous allocation ended.
import java.util.*;

public class NextFit {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of blocks: ");
        int m = sc.nextInt();

        int[] block = new int[m];
        System.out.println("Enter block sizes:");
        for (int i = 0; i < m; i++)
            block[i] = sc.nextInt();

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] process = new int[n], allocation = new int[n];

        System.out.println("Enter process sizes:");
        for (int i = 0; i < n; i++) {
            process[i] = sc.nextInt();
            allocation[i] = -1;
        }

        int last = 0;

        for (int i = 0; i < n; i++) {
            int count = 0;
            int j = last;

            while (count < m) {
                if (block[j] >= process[i]) {
                    allocation[i] = j;
                    block[j] -= process[i];
                    last = j;
                    break;
                }

                j = (j + 1) % m;
                count++;
            }
        }

        System.out.println("\nProcess\tSize\tBlock");
        for (int i = 0; i < n; i++) {
            System.out.print("P" + (i + 1) + "\t" + process[i] + "\t");
            if (allocation[i] != -1)
                System.out.println(allocation[i] + 1);
            else
                System.out.println("Not Allocated");
        }
    }
}


Instructions: Enter block sizes and process sizes when prompted. No extra file is needed.

Practical 10: Worst Fit Memory Allocation
Filename: WorstFit.java

Worst Fit allocates a process to the largest available memory block.
import java.util.*;

public class WorstFit {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of blocks: ");
        int m = sc.nextInt();

        int[] block = new int[m];
        System.out.println("Enter block sizes:");
        for (int i = 0; i < m; i++)
            block[i] = sc.nextInt();

        System.out.print("Enter number of processes: ");
        int n = sc.nextInt();

        int[] process = new int[n], allocation = new int[n];

        System.out.println("Enter process sizes:");
        for (int i = 0; i < n; i++) {
            process[i] = sc.nextInt();
            allocation[i] = -1;
        }

        for (int i = 0; i < n; i++) {
            int worst = -1;

            for (int j = 0; j < m; j++) {
                if (block[j] >= process[i]) {
                    if (worst == -1 || block[j] > block[worst])
                        worst = j;
                }
            }

            if (worst != -1) {
                allocation[i] = worst;
                block[worst] -= process[i];
            }
        }

        System.out.println("\nProcess\tSize\tBlock");
        for (int i = 0; i < n; i++) {
            System.out.print("P" + (i + 1) + "\t" + process[i] + "\t");
            if (allocation[i] != -1)
                System.out.println(allocation[i] + 1);
            else
                System.out.println("Not Allocated");
        }
    }
}


Instructions: Enter block sizes and process sizes when prompted. No extra file is needed.

Practical 11: FIFO Page Replacement
Filename: FIFOPageReplacement.java

FIFO (First In First Out) replaces the page that entered memory first.
import java.util.*;

public class FIFOPageReplacement {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of pages: ");
        int n = sc.nextInt();

        int[] pages = new int[n];
        System.out.println("Enter reference string:");
        for (int i = 0; i < n; i++)
            pages[i] = sc.nextInt();

        System.out.print("Enter number of frames: ");
        int f = sc.nextInt();

        if (f <= 0) {
            System.out.println("Number of frames must be positive.");
            return;
        }

        Queue<Integer> queue = new LinkedList<>();
        int faults = 0;

        System.out.println("\nPage\tFrames\t\tStatus");

        for (int page : pages) {
            boolean hit = queue.contains(page);

            if (!hit) {
                faults++;
                if (queue.size() == f)
                    queue.poll();
                queue.add(page);
            }

            System.out.println(page + "\t" + queue + "\t"
                    + (hit ? "Hit" : "Fault"));
        }

        System.out.println("\nTotal Page Faults: " + faults);
    }
}


Instructions: Enter the reference string one page number at a time, followed by the number of frames.

Practical 12: LRU Page Replacement
Filename: LRUPageReplacement.java

LRU (Least Recently Used) replaces the page that has not been used for the longest time.
import java.util.*;

public class LRUPageReplacement {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of pages: ");
        int n = sc.nextInt();

        int[] pages = new int[n];
        System.out.println("Enter reference string:");
        for (int i = 0; i < n; i++)
            pages[i] = sc.nextInt();

        System.out.print("Enter number of frames: ");
        int f = sc.nextInt();

        if (f <= 0) {
            System.out.println("Number of frames must be positive.");
            return;
        }

        List<Integer> frames = new ArrayList<>();
        int faults = 0;

        System.out.println("\nPage\tFrames\t\tStatus");

        for (int page : pages) {
            boolean hit = frames.contains(page);

            if (hit) {
                frames.remove(Integer.valueOf(page));
            } else {
                faults++;
                if (frames.size() == f)
                    frames.remove(0);
            }

            frames.add(page);

            System.out.println(page + "\t" + frames + "\t"
                    + (hit ? "Hit" : "Fault"));
        }

        System.out.println("\nTotal Page Faults: " + faults);
    }
}


Instructions: Enter the page reference string and number of frames. No extra file is required.

Practical 13: Optimal Page Replacement
Filename: OptimalPageReplacement.java

Optimal replacement replaces the page whose next use is farthest in the future.
import java.util.*;

public class OptimalPageReplacement {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of pages: ");
        int n = sc.nextInt();

        int[] pages = new int[n];
        System.out.println("Enter reference string:");
        for (int i = 0; i < n; i++)
            pages[i] = sc.nextInt();

        System.out.print("Enter number of frames: ");
        int f = sc.nextInt();

        if (f <= 0) {
            System.out.println("Number of frames must be positive.");
            return;
        }

        List<Integer> frames = new ArrayList<>();
        int faults = 0;

        System.out.println("\nPage\tFrames\t\tStatus");

        for (int i = 0; i < n; i++) {
            int page = pages[i];
            boolean hit = frames.contains(page);

            if (!hit) {
                faults++;

                if (frames.size() < f) {
                    frames.add(page);
                } else {
                    int farthest = -1;
                    int replace = -1;

                    for (int j = 0; j < frames.size(); j++) {
                        int next = Integer.MAX_VALUE;

                        for (int k = i + 1; k < n; k++) {
                            if (pages[k] == frames.get(j)) {
                                next = k;
                                break;
                            }
                        }

                        if (next > farthest) {
                            farthest = next;
                            replace = j;
                        }
                    }

                    frames.set(replace, page);
                }
            }

            System.out.println(page + "\t" + frames + "\t"
                    + (hit ? "Hit" : "Fault"));
        }

        System.out.println("\nTotal Page Faults: " + faults);
    }
}


Instructions: Enter the reference string and number of frames. No extra file is required.
