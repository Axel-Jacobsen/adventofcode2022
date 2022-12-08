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

function get_first(itr, pred::Function)
    # there must be a native julia fcn to do this
    for v in itr
        if pred(v)
            return v
        end
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



## Day 5!


```julia
function process_init_str_state(init_str_state)
    cleaned_strs = split(init_str_state, "\n")
    stack_strs = cleaned_strs[1:end-1, 1]
    numbers_str = cleaned_strs[end, 1]

    # get indexes of columns of boxes
    indexes::Vector{Int} = []
    for (i, col) in enumerate(numbers_str)
        if isdigit(col)
            push!(indexes, i)
        end
    end

    # create actual stacks of chars
    stacks = [Stack{Char}() for _ = 1:length(indexes)]
    for (j, index) in enumerate(indexes)
        for k = length(stack_strs):-1:1
            letter = stack_strs[k][index]
            if letter != ' '
                push!(stacks[j], letter)
            end
        end
    end
    stacks
end

function process_command_strs(command_strs)
    map(split(strip(command_strs), "\n")) do line
        # format is 'move n from x to y'
        num_strs = match(r"move (\d+) from (\d+) to (\d+)", line)
        (n, from, to) = map(s -> parse(Int64, s), num_strs.captures)
        (n, from, to)
    end
end

function process_d05_inputs()
    (initial_str_state, command_strs) =
        read("inputs/d05.txt", String) |> s -> split(s, "\n\n")

    stacks = initial_str_state |> process_init_str_state
    commands = command_strs |> process_command_strs
    stacks, commands
end

function peek_top_boxes(stacks)
    map(first, stacks) |> String
end


function p1()
    stacks, commands = process_d05_inputs()
    for command in commands
        n, from, to = command
        for i = 1:n
            push!(stacks[to], pop!(stacks[from]))
        end
    end
    peek_top_boxes(stacks)
end

function p2()
    stacks, commands = process_d05_inputs()
    for command in commands
        n, from, to = command
        from_vals = [pop!(stacks[from]) for _ = 1:n]
        for v in Iterators.reverse(from_vals)
            push!(stacks[to], v)
        end
    end
    peek_top_boxes(stacks)
end

p1(), p2()
```




    ("QGTHFZBHV", "MGDMPSZTM")



## Day 6!


```julia
function first_n_unique(str::String, n::Int)::Int
    chrs = Queue{Char}()
    for (i, letter) in enumerate(str)
        if length(chrs) == n
            dequeue!(chrs)
        end
        enqueue!(chrs, letter)
        if unique(chrs) |> length == n
            return i
        end
    end
    i
end

start_of_packet_idx(data) = first_n_unique(data, 4)
start_of_message_idx(data) = first_n_unique(data, 14)


function p1()
    data = read("inputs/d06.txt", String)
    start_of_packet_idx(data)
end

function p2()
    data = read("inputs/d06.txt", String)
    start_of_message_idx(data)
end

p1(), p2()
```




    (1855, 3256)



## Day 7!


```julia
struct File
    name::AbstractString
    size::Int
end

mutable struct Folder
    name::AbstractString
    parent::Union{Folder,Nothing}
    files::Vector{Union{File,Folder}}

    Folder(name::AbstractString) = new(name, nothing, Vector{File}())
    Folder(name::AbstractString, parent::Folder) = new(name, parent, Vector{File}())
    Folder(name::String, parent::Folder, files::Vector{File}) = new(name, files)
end


is_cmd = startswith("\$ ")
is_dir = startswith("dir ")
is_file = startswith(r"\d+")

function get_folder(folder_str::String, current_dir::Folder)::Folder
    name = split(folder_str, " ")[end]
    Folder(name, current_dir)
end

function get_file(file_str)::File
    (filesize_str, name) = split(file_str, " ")
    File(name, parse(Int, filesize_str))
end

function get_cmd(s)::Vector{String}
    cmd_vec = replace(s, "\$ " => "") |> s -> split(s, " ")
    if length(cmd_vec) == 1
        # if cmd doesn't have args, just set arg to be ""
        Vector{String}([cmd_vec[1], ""])
    else
        cmd_vec
    end
end

function execute_cd(folder::Folder, name::String)::Folder
    if name == ".."
        folder.parent
    else
        get_first(folder.files, f -> f.name == name)
    end
end

function process_terminal_command_history(day)
    data = process_inputs(day)
    num_lines = length(data)

    @assert (data[1] == "\$ cd /")

    root_folder = Folder("/")
    current_folder = root_folder

    i = 2
    next_line = data[i]
    while i <= num_lines
        @assert is_cmd(next_line)
        cmd, arg = get_cmd(next_line)

        if cmd == "ls"
            i += 1
            next_line = data[i]
            while !is_cmd(next_line)
                if is_dir(next_line)
                    push!(current_folder.files, get_folder(next_line, current_folder))
                elseif is_file(next_line)
                    push!(current_folder.files, get_file(next_line))
                else
                    @warn "unknown line in ls $(next_line)"
                end
                i += 1
                if i > num_lines
                    break
                end
                next_line = data[i]
            end
        elseif cmd == "cd"
            current_folder = execute_cd(current_folder, arg)
            i += 1
            next_line = data[i]
        else
            @warn "unknown cmd $(cmd)"
        end
    end
    root_folder
end

function get_folder_size(
    folder::Folder,
    subfolders::Union{Vector{Tuple{Folder,Int}},Nothing} = nothing,
)
    total_fs = mapreduce(+, folder.files) do fd
        if isa(fd, File)
            fd.size
        else
            folder_size, _ = get_folder_size(fd, subfolders)
            if subfolders != nothing
                push!(subfolders, (fd, folder_size))
            end
            folder_size
        end
    end

    if subfolders != nothing
        total_fs, subfolders
    else
        total_fs
    end
end

function p1()
    root_folder = process_terminal_command_history("07")
    root_folder_size, folder_sizes =
        get_folder_size(root_folder, Vector{Tuple{Folder,Int}}())

    s = 0
    for (folder, fs) in folder_sizes
        if fs <= 100000
            s += fs
        end
    end
    s
end

function p2()
    root_folder = process_terminal_command_history("07")
    root_folder_size, folder_sizes =
        get_folder_size(root_folder, Vector{Tuple{Folder,Int}}())

    mem_required_for_update = root_folder_size - (70000000 - 30000000)
    for (folder, fs) in sort(folder_sizes, by = x -> x[2])
        if fs > mem_required_for_update
            return fs
        end
    end
end

p1(), p2()
```




    (1644735, 1300850)


