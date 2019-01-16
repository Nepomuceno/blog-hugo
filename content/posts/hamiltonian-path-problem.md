---
title: The Hamiltonian path problem!
date: 2018-01-29 01:01:01
---

I was watching the video on [Numberfile](https://www.youtube.com/watch?v=G1m7goLCJDY) with Matt Parker he proposed a simple problem that you receive all the positive integers up to a certain number and you need to arrange them in a new order so that the sum of any 2 adjacent numbers will be a square number. Spoil alert now I will be giving a valid solution so if you are trying to solve that by yourself go there and try it (maybe the title has given you the idea of the solution already).

<!--more-->

Matt explains the problem much better than me, so for a better explanation. if you watch the [extra footage](https://youtu.be/7_ph5djCCnM) you would see that Charlie Turner shows that for the numbers greater than 24 util 299 all of them work and there is a conjecture that it would work for all numbers larger than 24.

Seeing that I just thought that "Wait 299 its a way small number". I know how to program this seams like a really easy program to make so why can I just prove this for something like prove there are solutions up to 500,000 and not only that get all the valid solutions up to 500,000.

The first thing that I checked was how hard was to get the [Hamiltonian path](https://en.wikipedia.org/wiki/Hamiltonian_path) in a graph and turns out that it is an [NP-Complete](https://en.wikipedia.org/wiki/NP-completeness) problem. This is not something that is really encouraging when you realize that the problem that you are trying to solve it is an NP-Complete problem. I thought that the fact that this is an NP-Complete problem for the solution of any graph doesn't mean that it is an NP-Complete problem for the solution of the graph in question we might have some tricks to do that would make solving this for the graph in question a bit easier than the general case. There are for example some sufficient conditions for the Hamiltonian path that might work in less than exponential time to check if the graphs have a Hamiltonian path without the need to actually find this path. Some examples of this tricks can be seen on this [awesome math stack exchange response](https://math.stackexchange.com/questions/130425/hamiltonian-path-detection)

So I decided to start with the simple things I need to generate this graphs a simple c# console app would solve that for me:

I am using the following data model (I know I used ulong when I could have used uint but I was really hopeful in the beginning) :

```csharp
    public struct HamiltonianGraph
    {
        public Dictionary<ulong, HashSet<ulong>> Nodes { get; set; }
        public int NumberOfVertices { get; set; }
        public int NumberOfEdges { get; set; }
        public bool HasSolution { get; set; }
        public ConcurrentBag<HashSet<ulong>> Solutions { get;set; }
    }
```

My basic program to generate the graphs was:

```csharp
    static void Main(string[] args)
    {
        ulong maxNumber = 5000;
        var graph = GenerateFiles(maxNumber);
        Console.WriteLine("The End!");
    }
    private static HamiltonianGraph GenerateFiles(ulong maxNumber)
    {
        HamiltonianGraph graph = new HamiltonianGraph();
        graph.Nodes = new Dictionary<ulong, HashSet<ulong>>();
        var lastNumberOfEdges = 0;
        for (ulong i = 1; i <= maxNumber; i++)
        {
            graph.Nodes.Add(i, new HashSet<ulong>());
            graph.NumberOfVertices++;
            for (ulong j = 1; j < i; j++)
            {
                if (IsSquare(i + j))
                {
                    graph.NumberOfEdges++;
                    graph.Nodes[i].Add(j);
                    graph.Nodes[j].Add(i);
                }
            }
            Console.WriteLine($"Vertice Added: {i.ToString("00000")} Number of Edges: {graph.NumberOfEdges} Edges per vertice: {(graph.NumberOfEdges / (graph.NumberOfVertices * 1.0)).ToString("F2")}, Edges added: {graph.NumberOfEdges - lastNumberOfEdges}");
            lastNumberOfEdges = graph.NumberOfEdges;
            File.WriteAllText($"cache/{i.ToString("00000000")}.json", JsonConvert.SerializeObject(savegraph));
        }
        return graph;
    }
    static bool IsSquare(ulong apositiveint)
    {
        return Math.Sqrt(apositiveint) % 1 == 0;
    }

```

As you can see by the number of zeros that I put on the file numbers I was really hopeful that I would be able to generate and test thousands of files. THee process of generating files was too slow and the files were getting really large especially after 7000 and since I don't have such a big hard drive I decided to stop there (I actually did some changes to save it to the cloud to m\ake it available to more people but we will talk about that in the next blog post)

Another improvement that I did in order to generate the files with the graphs was to get a better algorithm to verify that a number it is a square number. you can see some really clever ways to do this on this [stack overflow answer](https://stackoverflow.com/questions/295579/fastest-way-to-determine-if-an-integers-square-root-is-an-integer)

so after that, the code for `IsSquare` changed to:

```csharp
static bool IsSquare(ulong apositiveint)
{
    switch ((int)(apositiveint & 0x3F))
    {
        case 0x00:
        case 0x01:
        case 0x04:
        case 0x09:
        case 0x10:
        case 0x11:
        case 0x19:
        case 0x21:
        case 0x24:
        case 0x29:
        case 0x31:
        case 0x39:
            return Math.Sqrt(apositiveint) % 1 == 0;
        default:
            return false;
    }
}
```

was not the most optimized way to do it but was something that I could read easily and gave me most of the performance benefits.

Now that I had a way to generate the graphs and had them generated up to 7000 was the time to verify if there was a Hamiltonian path to each one of this graphs. As I said previously I like to start with the simplest solution: so I used what I know about backtracking and made a simple solution based on backtracking that would try a deep first approach

```csharp
private bool IsHamiltonian(ulong n, HashSet<ulong> sequence, Dictionary<ulong, HashSet<ulong>> currentNodes)
{
    //Adding the number the the current sequence
    sequence.Add(n);
    //if the sequence it is as big as the whole graph stop because we fould a solution
    if (sequence.Count == currentNodes.Count)
    {
        return true;
    }
    //Get all the nodes that I can reach from the current node
    HashSet<ulong> reachableNodes = currentNodes[n];
    //check the for each one of the intenal nodes if there will be a solution;
    foreach (var destinationNode in reachableNodes)
    {
        //if the path doesnt lead to a node already visited
        if (!sequence.Contains(destinationNode))
        {
            // recustivily check the solution to the rest of the nodes
            bool result = IsHamiltonian(destinationNode, sequence, currentNodes);
            //if there was a solution for this path returns
            if (result)
                return true;
        }
    }
    //if you arrive here this meand that this path it is nto valid so remove this node and returns false
    sequence.Remove(n);
    return false;
}
```

This is a basic implementation but that works. I would be more than happy to pass hours only talking about this simple algorithm and how it could be improved (I might do that in following posts).

The problem with this it is in order to prove that there is no Hamiltonian path you need to tray starting from all the vertices in your graph.

so the code to verify a graph was the following:

```csharp
public HamiltonianGraph VerifyHamilton(HamiltonianGraph graph)
{
    if (graph.Nodes.Count > 3)
    {
        foreach (var node in graph.Nodes)
        {
            var solution = new HashSet<ulong>();
            if (IsHamiltonian(node.Key, solution, graph.Nodes))
            {
                Console.WriteLine($"Solution for {graph.Nodes.Count} Starting at: {node.Key}, Solution: {string.Join(':', solution)}");
                graph.Solutions.Add(solution);
                graph.HasSolution = true;
                return graph;
            }
        }
    }
    return graph;
}
```

I got up to 80 (all proven to be correct) before I got tired of this slow solution and decided to try some optimization and do some things differently. But this is the base case. Let me know if you found interesting. I will be posting some optimizations and how you can leverage from cloud computing to scale that up for you in future posts.
