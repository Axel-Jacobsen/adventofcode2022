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



## Day 2!


```julia
ops = Dict("A" => 0, "X" => 0, "B" => 1, "Y" => 1, "C" => 2, "Z" => 2)

function rps(a, b)
    if (ops[a] + 0) % 3 == ops[b]
        3  # draw
    elseif (ops[a] + 1) % 3 == ops[b]
        0  # lose
    elseif (ops[a] + 2) % 3 == ops[b]
        6  # win
    end
end

rps_val(c) = ops[c] + 1

function p1()
    raw_data = process_inputs(s -> split(s, " "), "02")
    rps_value(pair) = rps(pair[2], pair[1]) + rps_val(pair[2])
    mapreduce(rps_value, +, raw_data)
end

function p2()
    throw_idxs = ["A", "B", "C"]
    result_matrix = [
        ["Y" "X" "Z"]
        ["Z" "Y" "X"]
        ["X" "Z" "Y"]
    ]
    raw_data = process_inputs(s -> split(s, " "), "02")

    function strategy(pair)
        their_throw = findfirst(s -> s == pair[1], throw_idxs)
        my_throw_idx = findfirst(s -> s == pair[2], result_matrix[:, their_throw])
        my_throw = throw_idxs[my_throw_idx]
        rps(my_throw, pair[1]) + rps_val(my_throw)
    end
    mapreduce(strategy, +, raw_data)
end

p1(), p2()
```




    (12772, 11618)


