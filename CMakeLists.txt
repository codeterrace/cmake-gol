cmake_minimum_required(VERSION 3.11)

# Global to hold the state of our random number generator
set_property(GLOBAL PROPERTY prng_state 1234)

# Basic linear congruential pseudo random number generator
# We'll use this PRNG to initialize the state of the game board
function(rand out)
  get_property(state GLOBAL PROPERTY prng_state)
  math(EXPR local_out "(1277*${state}+12345)%12397")
  set_property(GLOBAL PROPERTY prng_state ${local_out})
  set(${out} ${local_out} PARENT_SCOPE)
endfunction()

# from SO https://stackoverflow.com/questions/3307789/replacing-values-in-cmake-lists
macro(list_replace LIST INDEX NEWVALUE)
  list(INSERT ${LIST} ${INDEX} ${NEWVALUE})
  MATH(EXPR __INDEX "${INDEX} + 1")
  list (REMOVE_AT ${LIST} ${__INDEX})
endmacro(list_replace)

# The game board dimension is N tiles x N tiles
set(N 35)
math(EXPR N_MINUS_ONE "${N}-1")

# Initialize the game board -- set each randomly to alive or dead
macro(initialize)
  set(i 0)
  while(i LESS N)
    set(j 0)
    while(j LESS N)
      rand(rnum)
      MATH(EXPR val "${rnum}%8")
      if(i EQUAL 0 OR i EQUAL N_MINUS_ONE OR j EQUAL 0 OR j EQUAL N_MINUS_ONE)
        list(APPEND row_${i} 0)
      else()
        if(val GREATER 6) # If PRNG output > 6, the cell is alive
          list(APPEND row_${i} 1)
        else()
          list(APPEND row_${i} 0) # Else, the cell is dead
        endif()
      endif()
      MATH(EXPR j "${j}+1")
    endwhile()
    MATH(EXPR i "${i}+1")
  endwhile()
endmacro()

# Print out the game board
macro(print)
  set(i 0)
  while(i LESS N)
    set(r ${row_${i}})
    string(REPLACE "0" " " r_ ${r})
    string(REPLACE "1" "#" r__ ${r_})
    message(${r__})
    MATH(EXPR i "${i}+1")
  endwhile()
endmacro()

# Test to see if cell at coordinates i,j should be alive in the next generation
# According to the rules of conway's game of life:
#   - A live cell with < 2 live neighbours dies
#   - A live cell with 2 or 3 live neighbours lives
#   - A live cell with > 3 live neighbours dies
#   - A dead cell with exactly 3 live neighbours becomes alive
function(is_alive i j out)
  MATH(EXPR i_MINUS_ONE "${i}-1")
  MATH(EXPR i_PLUS_ONE "${i}+1")
  MATH(EXPR j_MINUS_ONE "${j}-1")
  MATH(EXPR j_PLUS_ONE "${j}+1")
  # Each cell has 8 neighbours, numbered v0 through v7
  list(GET row_${i} ${j_MINUS_ONE} v0)
  list(GET row_${i} ${j_PLUS_ONE} v1)
  list(GET row_${i_MINUS_ONE} ${j} v2)
  list(GET row_${i_MINUS_ONE} ${j_MINUS_ONE} v3)
  list(GET row_${i_MINUS_ONE} ${j_PLUS_ONE} v4)
  list(GET row_${i_PLUS_ONE} ${j_MINUS_ONE} v5)
  list(GET row_${i_PLUS_ONE} ${j_PLUS_ONE} v6)
  list(GET row_${i_PLUS_ONE} ${j} v7)
  # Add up the live neighbour count ...
  MATH(EXPR neighbours "${v0}+${v1}+${v2}+${v3}+${v4}+${v5}+${v6}+${v7}")
  list(GET row_${i} ${j} cur)
  if(cur EQUAL 1)
    if(neighbours EQUAL 2 OR neighbours EQUAL 3)
      set(${out} 1 PARENT_SCOPE)
    else()
      set(${out} 0 PARENT_SCOPE)
    endif()
  else()
    if(neighbours EQUAL 3)
      set(${out} 1 PARENT_SCOPE)
    else()
      set(${out} 0 PARENT_SCOPE)
    endif()
  endif()
  
endfunction()

# Update gameboard state
macro(update)
set(i 1)
while(i LESS N_MINUS_ONE)
  set(j 1)
  while(j LESS N_MINUS_ONE)
    is_alive(${i} ${j} res)
    list_replace(row_${i} ${j} ${res})
    MATH(EXPR j "${j}+1")
  endwhile()
  MATH(EXPR i "${i}+1")
endwhile()
endmacro()

initialize()

# Main loop
while(TRUE)
  update()
  print()
endwhile()

