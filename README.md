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

function process_inputs(convert::Function, day::String)
    open("inputs/d$day.txt", "r") do io
        map(s -> convert(s), eachline(io))
    end
end

process_inputs(day::String) = process_inputs(s -> s, day)
process_inputs(::Type{Int}, day::String) = process_inputs(s -> parse(Int64, s), day)

function matrix_inputs(day::String)
    process_inputs(day) do line
        # vector of ints
        map(s -> parse(Int, s), collect(line))
        # convert vec of vec of int to matrix of int
    end |> vv -> mapreduce(permutedims, vcat, vv)
end

clockmod(i::Int, m::Int) = i % m == 0 ? m : i % m

alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ"

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



## Day 3!


```julia
function p1()
    data = process_inputs(
        # split each line in half and take the common letter in each half
        s -> intersect(Set(s[1:length(s)÷2]), Set(s[length(s)÷2+1:end])),
        "03",
    )
    mapreduce(+, data) do sv
        # for each letter, get it's value
        letter = pop!(sv)
        findfirst(s -> s == letter, alphabet)
    end
end

group_n(arr, n::Int) = [arr[i:i+n-1] for i = 1:n:length(arr)]

function p2()
    # get each line of data for day 3 as a vector of strings
    data = process_inputs("03")
    elf_groups = group_n(data, 3)
    elf_group_ids = [pop!(intersect(g...)) for g in elf_groups]

    mapreduce(+, elf_group_ids) do letter
        findfirst(s -> s == letter, alphabet)
    end
end

p1(), p2()
```




    (8243, 2631)



## Day 4!


```julia
function get_d04_data()
    process_inputs("04") do str
        # parse eg string "5-7,4-9" into Vector{UnitRange}[5:7,4:9]
        elf_assignment_strs = split(str, ",")
        elf_assignment_ranges = map(elf_assignment_strs) do s
            (s0, s1) = split(s, "-")
            parse(Int64, s0):parse(Int64, s1)
        end
    end
end

function p1()
    quantify(get_d04_data()) do (r1, r2)
        # if the intersect is r1 or r2, r1 or
        # r2 is entirely overlaps the other
        intersect(r1, r2) in (r1, r2)
    end
end

function p2()
    quantify(get_d04_data()) do (r1, r2)
        # if the intersect is empty, r1 and
        # r2 do not overlap
        intersect(r1, r2) |> collect |> isempty |> Base.:!
    end
end

p1(), p2()
```




    (509, 870)


