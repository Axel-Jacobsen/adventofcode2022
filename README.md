# Advent of Code, 2022!

It is truly the most wonderful time of the year! Here is my work for this year's Advent.


```julia
using DataStructures
using JupyterFormatter
enable_autoformat()

# Helpers, of course
function quantify(predicate::Function, data)
    mapreduce(predicate, +, data)
end


function process_inputs(day::String)
    open("inputs/d$day.txt", "r") do io
        map(s -> parse(Int64, s), eachline(io))
    end
end

function process_inputs(convert::Function, day::String)
    open("inputs/d$day.txt", "r") do io
        map(s -> convert(s), eachline(io))
    end
end

function matrix_inputs(day::String)
    process_inputs(day) do line
        # vector of ints
        map(s -> parse(Int, s), collect(line))
        # convert vec of vec of int to matrix of int
    end |> vv -> mapreduce(permutedims, vcat, vv)
end

clockmod(i::Int, m::Int) = i % m == 0 ? m : i % m

cartdirs = ((-1, 0), (1, 0), (0, -1), (0, 1))
diagdirs = ((-1, 0), (1, 0), (0, -1), (0, 1), (-1, 1), (1, 1), (-1, -1), (1, -1))

Base.:+(p1::Tuple{Int,Int}, p2::Tuple{Int,Int}) = (p1[1] + p2[1], p1[2] + p2[2])
Base.:≤(p1::Tuple{Int,Int}, p2::Tuple{Int,Int}) = p1[1] ≤ p2[1] && p1[2] ≤ p2[2]
```

## Day 1!


```julia
function p1()
    raw_data = read("inputs/d01.txt", String) |> strip |> s -> split(s, "\n\n")

    chunks = map(s -> split(s, "\n"), raw_data)

    calories = map(chunks) do chunk
        mapreduce(s -> parse(Int64, s), +, chunk)
    end

    maximum(calories)
end


function p2()
    raw_data = read("inputs/d01.txt", String) |> strip |> s -> split(s, "\n\n")

    chunks = map(s -> split(s, "\n"), raw_data)

    calories = map(chunks) do chunk
        mapreduce(s -> parse(Int64, s), +, chunk)
    end |> sort

    calories[end-2:end] |> sum
end


p1(), p2()
```




    (68802, 205370)


