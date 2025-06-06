import java.util.*;
import java.util.stream.Collectors;

public class JobDependencyTreePrinter {

    // 模拟Job依赖关系的数据结构
    private static Map<Integer, JobNode> jobGraph = new HashMap<>();

    // Job节点类
    static class JobNode {
        int jobId;
        List<Integer> preJobs; // 前置Job列表
        List<Integer> postJobs; // 后置Job列表

        public JobNode(int jobId) {
            this.jobId = jobId;
            this.preJobs = new ArrayList<>();
            this.postJobs = new ArrayList<>();
        }
    }

    // 初始化Job依赖关系图（示例数据）
    static {
        // 添加Job依赖关系
        // 参数列表：第一个是当前Job，后面的是它的前置Job
        addJobDependency(1);  // job1没有前置
        addJobDependency(2, 1);  // job2依赖job1
        addJobDependency(3, 1);  // job3依赖job1
        addJobDependency(4, 2, 3);  // job4依赖job2和job3
        addJobDependency(5, 4);  // job5依赖job4
        addJobDependency(6, 5);  // job6依赖job5
        addJobDependency(7, 5, 8);  // job7依赖job5和job8
        addJobDependency(8);  // job8没有前置
    }

    // 添加Job依赖关系（使用可变参数）
    private static void addJobDependency(int jobId, int... preJobIds) {
        JobNode jobNode = jobGraph.computeIfAbsent(jobId, JobNode::new);
        
        for (int preJobId : preJobIds) {
            jobNode.preJobs.add(preJobId);
            
            // 同时维护后置Job关系
            JobNode preJobNode = jobGraph.computeIfAbsent(preJobId, JobNode::new);
            preJobNode.postJobs.add(jobId);
        }
    }

    // 反向打印Job依赖树（从根Job开始）
    public static void printReverseJobDependencyTree(int jobId) {
        if (!jobGraph.containsKey(jobId)) {
            System.out.println("Job '" + jobId + "' not found.");
            return;
        }

        System.out.println("Reverse dependency tree starting from root for job: " + jobId);
        System.out.println("(Note: '->' indicates job execution order)");
        System.out.println();

        // 找到所有根Job（没有前置Job的Job）
        List<Integer> rootJobs = findRootJobs(jobId);
        
        if (rootJobs.isEmpty()) {
            System.out.println("No root job found (circular dependency detected).");
            return;
        }

        Set<Integer> visited = new HashSet<>();
        for (int rootJobId : rootJobs) {
            printReverseTree(rootJobId, "", true, visited, jobId);
        }
    }

    // 找到影响指定Job的所有根Job
    private static List<Integer> findRootJobs(int targetJobId) {
        Set<Integer> visited = new HashSet<>();
        Set<Integer> rootJobs = new HashSet<>();
        findRootJobsRecursive(targetJobId, visited, rootJobs);
        return new ArrayList<>(rootJobs).stream().sorted().collect(Collectors.toList());
    }

    private static void findRootJobsRecursive(int jobId, Set<Integer> visited, Set<Integer> rootJobs) {
        if (visited.contains(jobId)) {
            return;
        }
        visited.add(jobId);

        JobNode jobNode = jobGraph.get(jobId);
        if (jobNode.preJobs.isEmpty()) {
            rootJobs.add(jobId);
        } else {
            for (int preJobId : jobNode.preJobs) {
                findRootJobsRecursive(preJobId, visited, rootJobs);
            }
        }
    }

    // 递归反向打印树状结构（从根Job开始）
    private static void printReverseTree(int jobId, String prefix, boolean isTail, Set<Integer> visited, int targetJobId) {
        System.out.println(prefix + (isTail ? "└── " : "├── ") + "job" + jobId + 
                         (jobId == targetJobId ? " (TARGET)" : ""));

        if (visited.contains(jobId)) {
            System.out.println(prefix + (isTail ? "    " : "│   ") + "└── (circular dependency detected)");
            return;
        }
        visited.add(jobId);

        JobNode jobNode = jobGraph.get(jobId);
        List<Integer> postJobs = jobNode.postJobs.stream()
                .sorted()
                .collect(Collectors.toList());

        // 只打印通向目标Job的路径
        postJobs = postJobs.stream()
                .filter(postJob -> leadsToTarget(postJob, targetJobId, new HashSet<>()))
                .collect(Collectors.toList());

        for (int i = 0; i < postJobs.size(); i++) {
            boolean isLast = (i == postJobs.size() - 1);
            printReverseTree(postJobs.get(i), 
                           prefix + (isTail ? "    " : "│   "), 
                           isLast, 
                           new HashSet<>(visited),
                           targetJobId);
        }
    }

    // 检查Job是否最终通向目标Job
    private static boolean leadsToTarget(int jobId, int targetJobId, Set<Integer> visited) {
        if (jobId == targetJobId) {
            return true;
        }
        if (visited.contains(jobId)) {
            return false;
        }
        visited.add(jobId);

        JobNode jobNode = jobGraph.get(jobId);
        for (int postJobId : jobNode.postJobs) {
            if (leadsToTarget(postJobId, targetJobId, new HashSet<>(visited))) {
                return true;
            }
        }
        return false;
    }

    public static void main(String[] args) {
        // 示例：反向打印job6的依赖树
        printReverseJobDependencyTree(6);
        
        System.out.println("\n----------------\n");
        
        // 示例：反向打印job7的依赖树
        printReverseJobDependencyTree(7);
    }
}
