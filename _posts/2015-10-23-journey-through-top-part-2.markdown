---
layout: post
title:  "Journey through TOP - Part 2 - Ruby"
date:   2015-10-23 13:30:00
categories: jekyll update
---
My journey through The Odin Project goes on, and I have just finished the second part focusing on the Ruby programming language.

This part has been a great challenge. I struggled many times, but at the end I learned a lot and I feel much more confident.

## Ruby Ruby Ruby Rubyyy

It begins with some simple programs to write, as a way to familiarize yourself with Ruby. You even have to rewrite some existing methods from the Enumerable class. And it's a great way to learn how Ruby works in the inside.
Then serious stuff begin. OOP, serialization, API interaction, recursion, data structures and TDD. But you never feel lost. The curriculum is really well made, you tackle all these subjects one at a time and going through each problem is an incredible source of motivation for braving the next one!

[picture with sword]

It took me 2 months to achieve this part, and it wasn't wasting time.

Let's talk about the final ruby project I realized, a command line chess game.

## Chess Game

In this final project, I had to create a player vs player chess game that you can play on the command line. You can see all the functionalities implemented in the following program that launches the chess game.

{% highlight ruby linenos%}
# Chess
#
# Player vs Player
# Script to launch a new game
#
# Functionalities implemented:
#
# - Prints "Check!" when a king is in check
#     The player in check has no choice but to move his king on a safe case, to move a
#     piece between his king and the adverse piece, or to take this adverse piece.
#
# - Prints "Checmate!" when a king is in checkmate and exit the game
#     Happens when the player's king is in check and has no solution to avoid the same
#     situation the following turn.
#
# - Prints "Draw!" if the game finishes in a draw
#     When only the kings remain on the board.
#
# - Prints "Slatemate!" if the game finishes with a slatemate
#     When the player has got only his king remaining on the board, and can't move it
#     without putting it in check.
#
# - Pawn can double case when at initial location
#
# - Pawn can be changed in another piece when reaches opposite side
#     Piece selected by player.
#
# - Castling can be made
#     If the player's king is not in check and can't be on the way to castle. And if the
#     king and the rook concerned have not moved since beginning of game.
#
# - Game can be saved at beginning of every turn. If so, exit the game. Player chooses to
#   load a game or create a new one at beginning of game.

require './lib/game.rb'

game = Game.new
game.start
{% endhighlight %}

Basically, all the usual functionalities of a chess game have been implemented!

I'm a chess player, so I wanted to be able to play chess with the true rules. I would have found it really annoying if I couldn't castle because it's part of the strategy of the game.
Now, let's have a look at the design.

## OOP, check!

Before doing this chess game, I already had to create some command line games, like a Tic Tac Toe or a Connect Four. So I knew how I wanted to design my chess game. 

This one would be divided in four classes:

* Player class

{% highlight ruby linenos%}
# Class for the players. Their name and which color they play.
class Player
  attr_reader :name, :color

  def initialize (name, color)
    @name = name
    @color = color
  end
end
{% endhighlight %}

The Player class is really basic. Its purpose is to store the player's name and the color he plays, white or black.

<br>
* Board class

{% highlight ruby linenos%}
# Class for the board. Allows to display the chess board and to get and set squares.
class Board
  attr_reader :grid

  def initialize(grid = nil)
    @grid = grid || default_grid
  end

  # Fill the board with all the pieces in their intitial location
  def default_grid
    array = Array.new(8) { Array.new(8) }

    array[0][0] = Rook.new('white', [0,0], 'slide')
    array[1][0] = Knight.new('white', [1,0], 'step')
    array[2][0] = Bishop.new('white', [2,0], 'slide')
    array[3][0] = Queen.new('white', [3,0], 'slide')
    array[4][0] = King.new('white', [4,0], 'step')
    array[5][0] = Bishop.new('white', [5,0], 'slide')
    array[6][0] = Knight.new('white', [6,0], 'step')
    array[7][0] = Rook.new('white', [7,0], 'slide')
    array[0..7].each_with_index { |column, index| 
                                   column[1] = Pawn.new('white', [index,1], 'step') }

    array[0][7] = Rook.new('black', [0,7], 'slide')
    array[1][7] = Knight.new('black', [1,7], 'step')
    array[2][7] = Bishop.new('black', [2,7], 'slide')
    array[3][7] = Queen.new('black', [3,7], 'slide')
    array[4][7] = King.new('black', [4,7], 'step')
    array[5][7] = Bishop.new('black', [5,7], 'slide')
    array[6][7] = Knight.new('black', [6,7], 'step')
    array[7][7] = Rook.new('black', [7,7], 'slide')
    array[0..7].each_with_index { |column, index| 
                                   column[6] = Pawn.new('black', [index,6], 'step') }

    array
  end

  # Display the board
  def display
    puts
    puts "   |---------------------------------------|"
    8.downto(1) do |row|
      print " #{row} | "
      8.times do |col|
        print grid[col][row-1] ? grid[col][row-1].unicode.encode('utf-8') : ' '
        print '  | '
      end
      puts
      puts "   |----|----|----|----|----|----|----|----|"
    end
    puts "     A    B    C    D    E    F    G    H"
    puts
  end

  # Return the piece selected
  def get_case(case_selected)
    grid[case_selected[0]][case_selected[1]]
  end

  # Fill the case given by the piece given
  def set_case(case_selected, piece)
    grid[case_selected[0]][case_selected[1]] = piece
    unless piece.nil?
      piece.location = case_selected
    end
  end
end
{% endhighlight %}

The Board class aim is to create a chess board and be able to display it. the <code class="highlight">get_case</code> and <code class="highlight">set_case</code> methods allow to read and update a case from a Board object.

<br>
* Pieces class

{% highlight ruby linenos%}
# Parent class of all pieces.
class Pieces
  attr_reader :unicode, :color, :type
  attr_accessor :location, :path, :counter

  def initialize (color, location, type, counter = 0)
    @color = color
    @location = location
    @type = type
    @counter = counter
  end
end

class King < Pieces
  def initialize (color, location, type, counter = 0)
    super
    @color == 'white' ? @unicode = "\u2654" : @unicode = "\u265A"
  end

  def possible_moves
    moves = [[-1, 0], [-1, 1], [ 0, 1], [ 1, 1],
             [ 1, 0], [ 1,-1], [ 0,-1], [-1,-1],
             [-2, 0], [ 2, 0]]
    moves
  end
end

class Queen < Pieces
  def initialize (color, location, type, counter = 0)
    super
    @color == 'white' ? @unicode = "\u2655" : @unicode = "\u265B"
    @path = []
  end

  def possible_moves
    moves = [[-1, 0], [-1, 1], [ 0, 1], [ 1, 1],
             [ 1, 0], [ 1,-1], [ 0,-1], [-1,-1]]
    moves
  end
end

class Rook < Pieces
  def initialize (color, location, type, counter = 0)
    super
    @color == 'white' ? @unicode = "\u2656" : @unicode = "\u265C"
    @path = []
  end

  def possible_moves
    moves = [[-1, 0], [ 0, 1], [ 1, 0], [ 0,-1]]
    moves
  end
end

class Bishop < Pieces
  def initialize (color, location, type, counter = 0)
    super
    @color == 'white' ? @unicode = "\u2657" : @unicode = "\u265D"
    @path = []
  end

  def possible_moves
    moves = [[-1, 1], [ 1, 1], [ 1,-1], [-1,-1]]
    moves
  end
end

class Knight < Pieces
  def initialize (color, location, type, counter = 0)
    super
    @color == 'white' ? @unicode = "\u2658" : @unicode = "\u265E"
  end

  def possible_moves
    moves = [[-2,-1], [-2, 1], [-1, 2], [ 1, 2],
             [ 2, 1], [ 2,-1], [-1,-2], [ 1,-2]]
    moves
  end
end

class Pawn < Pieces
  def initialize (color, location, type, counter = 0)
    super
    @color == 'white' ? @unicode = "\u2659" : @unicode = "\u265F"
  end

  def possible_moves
    if color == 'white'
      moves = [[-1, 1], [ 0, 1], [ 1, 1], [ 0, 2]]
    else
      moves = [[ 1,-1], [ 0,-1], [-1,-1], [ 0,-2]]
    end
    moves
  end
end
{% endhighlight %}

The Pieces class is a parent class for all the chess pieces. I initialized the attributes and some instances in it, avoiding repeatition in the following nested classes (DRY, very important!).
In the pieces classes are stored the unicodes, and a method that returns the way the pieces move on a board.
<br>
* Game class

The heart of the game. Let's see it in more details.

## Game class

{% highlight ruby linenos%}
require_relative 'board.rb'
require_relative 'pieces.rb'
require_relative 'player.rb'
require 'yaml'

# Main class with all the logic of the chess
class Game
  attr_accessor :current_player, :other_player, :board

  def initialize
    @board = Board.new
  end

  # Launch chess game.
  # Exit only if one player won, if there is a draw or if there is stalemate.
  def start
    welcome_players
    board.display
    until victory? || draw? || stalemate?
      next_turn
    end
  end



  ##################
  # WELCOME PLAYERS
  ##################
  
  # Possibility to load a saved game
  # Otherwise Ask players names and which color they want to play
  def welcome_players
    puts
    puts "Welcome to this chess game!"
    puts "Load a game? 'y' to load:"
    Dir.mkdir("save") unless Dir.exists? "save"

    choice = gets.chomp.downcase
    if choice.include? "y"
      load_game
      return
    end

    puts "Give me the name of the player who wants to be White:"
    @current_player = Player.new(gets.chomp.downcase, 'white')
    puts "Give me the name of the player who wants to be Black:"
    @other_player = Player.new(gets.chomp.downcase, 'black')
    puts "Ok let's start!"
  end



  ############
  # NEXT TURN
  ############
  # Play next player's turn:
  # The player has to select the case from which he wants to move the piece.
  # Then, he has to select the case where he wants to move the piece.
  # If the move is authorized, the piece is moved, the board displayed
  # and the other player becomes the current one.
  # 'C' allows the player to start again his turn.
  def next_turn
    puts
    puts "#{current_player.name.capitalize}, your turn. Select a piece or 'save' to save:"
    case_from = select_case
    save_game if case_from == 'save'
    return if case_from == 'C' || case_not_valid?(case_from, 'from')

    puts "Where do you want to move it? 'C' to select another piece."
    case_to = select_case
    return if case_to == 'C' || case_not_valid?(case_to, 'to') || case_to == 'save'

    answer = move_possible(case_from, case_to)
    if !answer[0]
      puts answer[1]
      return
    end
    move_piece(case_from, case_to) if board.get_case(case_from)  # If castling hasn't been done
    board.get_case(case_to).counter += 1  # Memorize number of times a piece moved

    board.display
    change_players
  end

  # Change order of players
  def change_players
    temp = self.current_player
    self.current_player = self.other_player
    self.other_player = temp
  end

  # Return the selected case by the current player in an array
  # ex: e4 => [4,3]
  def select_case
    selection = mapping(gets.chomp.downcase)
    unless selection
      puts "Sorry wrong input. Try again: (ex: e4)"
      selection = select_case
    end
    return selection
  end

  # Map the selected case to the actual board case
  def mapping(input)
    mapping = {
      'a1'=>[0,0], 'a2'=>[0,1], 'a3'=>[0,2], 'a4'=>[0,3], 'a5'=>[0,4], 'a6'=>[0,5],
      'a7'=>[0,6], 'a8'=>[0,7],
      'b1'=>[1,0], 'b2'=>[1,1], 'b3'=>[1,2], 'b4'=>[1,3], 'b5'=>[1,4], 'b6'=>[1,5],
      'b7'=>[1,6], 'b8'=>[1,7],
      'c1'=>[2,0], 'c2'=>[2,1], 'c3'=>[2,2], 'c4'=>[2,3], 'c5'=>[2,4], 'c6'=>[2,5],
      'c7'=>[2,6], 'c8'=>[2,7],
      'd1'=>[3,0], 'd2'=>[3,1], 'd3'=>[3,2], 'd4'=>[3,3], 'd5'=>[3,4], 'd6'=>[3,5],
      'd7'=>[3,6], 'd8'=>[3,7],
      'e1'=>[4,0], 'e2'=>[4,1], 'e3'=>[4,2], 'e4'=>[4,3], 'e5'=>[4,4], 'e6'=>[4,5],
      'e7'=>[4,6], 'e8'=>[4,7],
      'f1'=>[5,0], 'f2'=>[5,1], 'f3'=>[5,2], 'f4'=>[5,3], 'f5'=>[5,4], 'f6'=>[5,5],
      'f7'=>[5,6], 'f8'=>[5,7],
      'g1'=>[6,0], 'g2'=>[6,1], 'g3'=>[6,2], 'g4'=>[6,3], 'g5'=>[6,4], 'g6'=>[6,5],
      'g7'=>[6,6], 'g8'=>[6,7],
      'h1'=>[7,0], 'h2'=>[7,1], 'h3'=>[7,2], 'h4'=>[7,3], 'h5'=>[7,4], 'h6'=>[7,5],
      'h7'=>[7,6], 'h8'=>[7,7],
      'c' => 'C', 'save' => 'save'
    }
    mapping[input]
  end

  # Return true if the player is not allowed to play the selected case
  def case_not_valid?(case_selected, input)
    piece = board.get_case(case_selected)

    # case_from
    if input == 'from'
      if piece.nil?
        puts "There is no piece on this case. Start again."
        return true
      elsif piece.color != current_player.color
        puts "This is not your piece! Start again."
        return true
      end
    # case_to
    elsif input == 'to'
      if piece.nil?
        return false
      elsif piece.color == current_player.color
        puts "This is your piece! Start again."
        return true
      end
    end
    return false
  end

  # Return an array of 2 elements:
  # - a boolean, true if the piece can move from and to the selected case
  # - a string describing the error if piece can't move
  def move_possible(case_from, case_to)
    piece = board.get_case(case_from)
    move = [case_to[0] - case_from[0], case_to[1] - case_from[1]]

    # Check if castling 
    if piece.is_a?(King)
      if move == [-2, 0] || move == [2, 0]
        if can_castle?(piece, move)
          answer = [true]
        else
          answer = [false, "Can't castle sorry."]
        end
        return answer
      end 
    end

    # Check if move is coherent with piece's way of displacement
    if piece.type == 'step' && !piece.possible_moves.include?(move)
      answer = [false, "This piece can't move like that! Start again."]
      return answer
    end

    # Check if obstruction on path for sliding pieces
    if piece.type == 'slide' && obstruction?(piece, case_from, case_to)
      answer = [false, "Obstruction in the path. Start again."]
      return answer
    end

    # Check if a pawn can go in diag and can move up to 2 cases
    if piece.is_a?(Pawn)
      answer = [false, "Pawn can't move like that."]
      if move[0] != 0
        return answer if pawn_cant_diag(piece, case_to)
      elsif move[1] == 2 || move[1] == -2
        return answer if pawn_cant_double(piece, case_from)
      end
    end

    # Temporary move for the next tests
    board.set_case(case_from, nil)
    temp_piece = board.get_case(case_to)
    board.set_case(case_to, piece)
    answer = [true]

    # Check if kings stand one case apart
    if kings_too_close?
      answer = [false, "Kings can't be that close. Start again."]
    end

    # Check if king of current's player would be in check
    if king_check?(find_king(current_player))
      answer = [false, "If you do that, your king will be in check! Start again."]
    end

    # Come back to previous state before temporary move
    board.set_case(case_from, piece)
    board.set_case(case_to, temp_piece)
    return answer
  end

  # Move the piece to the selected case
  def move_piece(case_from, case_to)
    piece = board.get_case(case_from)

    # Case when pawn reaches last line
    piece = change_pawn(piece) if pawn_reached_end?(piece, case_to)

    board.set_case(case_to, piece)
    board.set_case(case_from, nil)
  end

  # Return true if there are obstacles on the path of a sliding piece
  def obstruction?(piece, case_from, case_to)
    piece.possible_moves.each do |coord|
      next_case = case_from
      loop do
        next_case = [next_case[0] + coord[0], next_case[1] + coord[1]]
        return false if next_case == case_to
        break if offboard(next_case) || !empty?(next_case)
      end
    end
  end

  # Return true if pawn reached the last line
  def pawn_reached_end?(piece, case_to)
    if piece.is_a?(Pawn)
      return true if case_to[1] == 0 || case_to[1] == 7
    end
    return false
  end

  # Ask the player if he wants to change the pawn in another piece and return it
  def change_pawn(piece)
    puts "Do you want to change the piece in another piece?"
    puts "'Q' for Queen, 'R' for Rook, 'B' for Bishop, 'K' for knight"
    puts "or 'P' if you want to keep a Pawn"
    loop do
      input = gets.chomp.upcase
      case input
      when 'Q'
        new_piece = Queen.new(piece.color, piece.location, 'slide')
      when 'R'
        new_piece = Rook.new(piece.color, piece.location, 'slide')
      when 'B'
        new_piece = Bishop.new(piece.color, piece.location, 'slide')
      when 'K'
        new_piece = Knight.new(piece.color, piece.location, 'slide')
      when 'P'
        new_piece = piece
      else
        puts "Wrong input. Try again"
        next
      end
      return new_piece
    end 
  end

  # Return true if pawn can't move in diag
  def pawn_cant_diag(piece, case_to)
    return true if empty?(case_to)
    case_selected = board.get_case(case_to)
    return true if case_selected.color == piece.color
    return false
  end

  # Return true if pawn can move up to 2 cases
  def pawn_cant_double(piece, case_from)
    if piece.color == 'white'
      return true if case_from[1] != 1
    elsif piece.color == 'black'
      return true if case_from[1] != 6
    end
    return false
  end

  # Return true if the case selected is free of pieces
  def empty?(case_selected)
    if board.get_case(case_selected) == nil
      return true
    end
    return false
  end

  # Return true if the case selected is offboard
  def offboard(coord)
    return true if coord[0] < 0 || coord[0] > 7 ||
                   coord[1] < 0 || coord[1] > 7
  end



  #############################
  # VICTORY, DRAW OR SLATEMATE
  #############################

  # Return true if one player won
  def victory?
    king = find_king(current_player)
    the_bad = king_check?(king)
    if the_bad
      if king_checkmate?(king, the_bad)
        puts "Checkmate!"
        puts "Well done #{other_player.name} you won!"
        return true
      else
        puts "Check!"
      end
    end
    return false
  end

  # Return true if there is a draw
  def draw?
    counter = 0
    board.grid.each do |col|
      col.each do |cell|
        counter +=1 if cell
      end
    end
    if counter < 3
      puts "Draw!"
      return true
    end
    return false
  end

  # Return true if there is a stalemate
  def stalemate?
    pieces = find_pieces(current_player.color)
    if pieces.size == 1
      unless king_can_move?(pieces[0])
        puts "Slatemate!"
        return true 
      end
    end
    return false
  end

  # Return king of given player
  def find_king(player)
    board.grid.each do |col|
      col.each do |cell|
        if cell
          return cell if cell.is_a?(King) && cell.color == player.color
        end
      end
    end
  end

  # Return an array with all the pieces of the given color
  def find_pieces(color)
    array = []
    board.grid.each do |column|
      column.each do |piece|
        next if !piece
        array.push(piece) if piece.color == color
      end
    end 
    array
  end

  # Return true if the given king is in check
  def king_check?(king)
    x = king.location[0]
    y = king.location[1]
    return check_slide?(x, y, 'diag') || check_slide?(x, y, 'line') ||
           check_step?(x, y, 'knight') || check_step?(x, y, 'pawn')
  end

  # Return true if king is in check by a sliding piece
  def check_slide?(x, y, input)
    diag = [[-1, 1], [1, 1], [1,-1], [-1,-1]]
    line = [[ 0, 1], [ 1, 0], [ 0,-1], [-1, 0]]

    input == 'diag' ? direction = diag : direction = line

    direction.each do |coord|
      next_case = [x, y]
      path = []
      loop do
        next_case = [next_case[0] + coord[0], next_case[1] + coord[1]]
        break if offboard(next_case)
        if empty?(next_case)
          path.push(next_case)
          next 
        end
        piece = board.get_case(next_case)
        if piece.color == other_player.color
          if direction == diag
            if piece.is_a?(Queen) || piece.is_a?(Bishop)
              piece.path = path
              return piece 
            end
          elsif direction == line
            if piece.is_a?(Queen) || piece.is_a?(Rook)
            piece.path = path
            return piece 
          end
          end
        else
          break
        end
      end
    end
    return false
  end

  # Return true if king is in check by a stepping piece
  def check_step?(x, y, input)
    knight = [[-2,-1], [-2, 1], [-1, 2], [ 1, 2],
              [ 2, 1], [ 2,-1], [ 1,-2], [-1,-2]]
    pawn = [[-1, 1], [ 1, 1]] if current_player.color == 'white'
    pawn = [[-1,-1], [ 1,-1]] if current_player.color == 'black'

    input == 'knight' ? direction = knight : direction = pawn

    direction.each do |coord|
      next_case = [coord[0] + x, coord[1] + y]
      next if offboard(next_case)
      next if empty?(next_case)
      piece = board.get_case(next_case)
      if piece.color == other_player.color
        return piece if piece.is_a?(Knight) && direction == knight
        return piece if piece.is_a?(Pawn) && direction == pawn
      else
        next
      end
    end
    return false
  end

  # Return true if the given king is checkmate
  def king_checkmate?(king, the_bad)
    # Not checkmate if king can move
    return false if king_can_move?(king)

    # Not checkmate if adverse piece can be taken or path obstructed
    if the_bad.type == 'step'
      return false if can_be_taken?(the_bad)
    elsif the_bad.type == 'slide'
      return false if can_be_taken?(the_bad) || can_be_obstructed?(the_bad)
    end

    return true
  end

  # Return true if there is at least one empty case around the king
  def king_can_move?(king)
    current_case = king.location
    king.possible_moves[0..-3].each do |coord|  # [0..-3] to avoid castle moves
      next_case = [current_case[0] + coord[0], current_case[1] + coord[1]]
      next if offboard(next_case)
      if empty?(next_case)
        move_piece(current_case, next_case)
        if king_check?(king)
          move_piece(next_case, current_case)
          next
        else
          move_piece(next_case, current_case)
          return true
        end
      end
    end
    return false
  end

  # Return true if piece can be taken by adverse piece
  def can_be_taken?(the_bad)
    the_bad.color == 'white' ? color = 'black' : color = 'white'
    case_to = the_bad.location
    pieces = find_pieces(color)
    pieces.each do |piece|
      case_from = piece.location
      answer = move_possible(case_from, case_to)
      return true if answer[0]
    end
    return false
  end

  # Return true if piece can be obstructed by adverse piece
  def can_be_obstructed?(the_bad)
    the_bad.color == 'white' ? color = 'black' : color = 'white'
    pieces = find_pieces(color)
    the_bad.path.each do |case_to|
      pieces.each do |piece|
        case_from = piece.location
        answer = move_possible(case_from, case_to)
        return true if answer[0]
      end
    end
    return false
  end

  # Return true if kings are one case apart
  def kings_too_close?
    king1 = find_king(current_player)
    king2 = find_king(other_player)
    king1.possible_moves.each do |coord|
      next_case = [king1.location[0] + coord[0], king1.location[1] + coord[1]]
      return true if next_case == king2.location
    end
    return false
  end

  # Return true if king can castle
  def can_castle?(king, move)
    if move == [-2, 0]
      path = [[4,0], [3,0], [2,0], [1,0], [0,0]] if king.color == 'white'
      path = [[4,7], [3,7], [2,7], [1,7], [0,7]] if king.color == 'black'
    elsif move == [ 2, 0]
      path = [[4,0], [5,0], [6,0], [7,0]] if king.color == 'white'
      path = [[4,7], [5,7], [6,7], [7,7]] if king.color == 'black'
    end

    piece1 = board.get_case(path.first)
    piece2 = board.get_case(path.last)

    # King and Rook should be at their initial location
    return false if empty?(path.first) || empty?(path.last)

    # Squares in between should be empty
    path[1..-2].each { |square| return false unless empty?(square) }

    # King & Rook shouldn't have mooved
    return false if piece1.counter > 0 || piece2.counter > 0
    
    # King shouldn't be in check on the path
    2.times do |nb|
      move_piece(path[nb], path[nb+1])
      if king_check?(king)
        move_piece(path[nb+1], path[0])
        return false
      end
    end
    move_piece(path.last, path[1])

    return true
  end



  ################################
  # LOAD AND SAVE FUNCTIONALITIES
  ################################

  # Load saved game if exists
  def load_game
    Dir.chdir("save")
    if File.exists?("save.txt")
      data = YAML::load(File.read("save.txt"))
      self.current_player = data.current_player
      self.other_player = data.other_player
      self.board = data.board
      puts "Game Loaded!"
      puts 
      Dir.chdir("..")
    else
      puts "You have no saved games."
      puts ""
      Dir.chdir("..")
    end
  end

  # Save game
  def save_game
    Dir.chdir("save")
    File.open("save.txt", 'w') { |file| file.write(YAML::dump(self))}
    puts "Game Saved!"
    Dir.chdir("..")
    exit
  end
end
{% endhighlight %}

The method <code class="highlight">#start</code> is the one used to launch a new game. From there, the player can choose to load a game or start a new one. If he starts a new one, he will be asked his name and which color he wants to play. Same for other player. Then we have a loop that allows the program to play the same instructions till we've got a winner, a draw or a slatemate (equals a draw).

This class is really long so I won't describe everything. Regards to the design, I tried to keep methods short and with one purpose. It's not always easy but I thing it really pays off. It makes the code cleaner, tidy, and much more easily to read. Furthermore I try to keep as much comments as possible, without disturbing the clearness of the code.

## Writing Specs, Check!

Using RSpec, I wrote some tests for the main methods of my Game class. And I really have to confess: testing seemed first a waste of time. But you see quickly how efficient it can be. These tests allowed me to correct some glitches in my code I wouldn't have found without. Or maybe I would, but 25 games later! So it's definitly not a waste of time.

{% highlight ruby linenos%}
require './lib/game.rb'

describe Game do

  let (:game) { Game.new }
  let (:flo) { Player.new("flo", "white") }
  let (:ginny) { Player.new("ginny", "black") }

  describe '#case_not_valid?' do
    context "when input is 'from'" do
      let (:input) { 'from'}
      it "returns false if a selected case is valide" do
        case_selected = [1,1]
        game.stub(:current_player) { flo }
        expect(game.case_not_valid?(case_selected, input)).to eq false
      end
      it "returns true if a case is empty" do
        case_selected = [3,3]
        game.stub(:current_player) { flo }
        expect(game.case_not_valid?(case_selected, input)).to eq true
      end
      it "returns true if the case is taken by an adverse piece" do
        case_selected = [7,7]
        game.stub(:current_player) { flo }
        expect(game.case_not_valid?(case_selected, input)).to eq true
      end
    end

    context "when input is 'to'" do
      let (:input) { 'to'}
      it "returns false if a selected case is valide" do
        case_selected = [3,3]
        expect(game.case_not_valid?(case_selected, input)).to eq false
      end
      it "returns true if the case is already taken by own piece" do
        case_selected = [1,1]
        game.stub(:current_player) { flo }
        expect(game.case_not_valid?(case_selected, input)).to eq true
      end
    end
  end


  describe '#move_possible' do
    context "regards to piece's way of displacement" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[0][0] = Rook.new('white', [0,0], 'slide')
        array[1][0] = Knight.new('white', [1,0], 'step')
        array[2][0] = Bishop.new('white', [2,0], 'slide')
        array[3][0] = Queen.new('white', [3,0], 'slide')
        array[4][0] = King.new('white', [4,0], 'step')
        array[5][1] = Pawn.new('white', [5,1], 'step')
        array[7][7] = King.new('black', [7,7], 'step')
        Board.new(array)
      end
      it "returns an array with 1st element eq to true if rook can move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([0,0], [0,1]).first).to eq true
      end
      it "returns an array with 1st element eq to false if rook can't move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([0,0], [1,1]).first).to eq false
      end
      it "returns an array with 1st element eq to true if knight can move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([1,0], [0,2]).first).to eq true
      end
      it "returns an array with 1st element eq to false if knight can't move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([1,0], [0,1]).first).to eq false
      end
      it "returns an array with 1st element eq to true if bishop can move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([2,0], [3,1]).first).to eq true
      end
      it "returns an array with 1st element eq to false if bishop can't move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([2,0], [2,1]).first).to eq false
      end
      it "returns an array with 1st element eq to true if queen can move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([3,0], [3,3]).first).to eq true
      end
      it "returns an array with 1st element eq to false if queen can't move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([3,0], [4,2]).first).to eq false
      end
      it "returns an array with 1st element eq to true if king can move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([4,0], [4,1]).first).to eq true
      end
      it "returns an array with 1st element eq to false if king can't move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([4,0], [4,2]).first).to eq false
      end
      it "returns an array with 1st element eq to true if pawn can move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([5,1], [5,2]).first).to eq true
      end
      it "returns an array with 1st element eq to false if pawn can't move" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([5,1], [6,1]).first).to eq false
      end
    end

    context "regards to sliding pieces" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[3][1] = Rook.new('white', [3,1], 'slide')
        array[3][2] = Bishop.new('white', [3,2], 'slide')
        array[4][1] = Queen.new('white', [4,1], 'slide')
        array[7][7] = King.new('black', [7,7], 'step')
        Board.new(array)
      end
      it "returns an array with 1st element eq to false if obstacle on the path" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([3,1], [5,1]).first).to eq false
        expect(game.move_possible([3,2], [5,0]).first).to eq false
        expect(game.move_possible([4,1], [2,1]).first).to eq false
      end
    end
    
    context "regards to the pawn" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[0][0] = King.new('white', [0,0], 'step')
        array[2][1] = Pawn.new('white', [2,1], 'step')
        array[3][2] = Pawn.new('white', [3,2], 'step')
        array[4][3] = Pawn.new('black', [4,3], 'step')
        array[7][7] = King.new('black', [7,7], 'step')
        Board.new(array)
      end
      it "returns an array with 1st element eq to true if pawn can double displacement" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([2,1], [2,3]).first).to eq true
      end
      it "returns an array with 1st element eq to true if pawn can go in diag" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([3,2], [4,3]).first).to eq true
      end
      it "returns an array with 1st element eq to false if pawn can't double displacement" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([3,2], [3,4]).first).to eq false
      end
      it "returns an array with 1st element eq to false if pawn can't go in diag" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([2,1], [1,2]).first).to eq false
      end
    end

    context "regards to kings location" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[3][3] = King.new('white', [3,3], 'step')
        array[3][5] = King.new('black', [3,5], 'step')
        Board.new(array)
      end
      it "returns an array with 1st element eq to false if kings are one case apart" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([3,5], [3,4]).first).to eq false
      end
    end

    context "regards to own king" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[0][0] = King.new('white', [0,0], 'step')
        array[7][7] = King.new('black', [7,7], 'step')
        array[6][0] = Queen.new('white', [6,0], 'slide')
        Board.new(array)
      end
      it "returns an array with 1st element eq to false if king becomes in check" do
        game.stub(:board) { board }
        game.stub(:current_player) { ginny }
        game.stub(:other_player) { flo }
        expect(game.move_possible([7,7], [6,7]).first).to eq false
      end
    end

    context "regards to castling, left bottom" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[0][0] = Rook.new('white', [0,0], 'slide')
        array[7][0] = Rook.new('white', [7,0], 'slide')
        array[4][7] = King.new('black', [4,7], 'step')
        array[0][7] = Rook.new('black', [0,7], 'slide')
        array[7][7] = Rook.new('black', [7,7], 'slide')
        Board.new(array)
      end
      it "returns an array with 1st element eq to true if king can castle" do
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        game.stub(:board) { board }
        expect(game.move_possible([4,0], [2,0]).first).to eq true
      end
    end
    context "regards to castling, right bottom" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[0][0] = Rook.new('white', [0,0], 'slide')
        array[7][0] = Rook.new('white', [7,0], 'slide')
        array[4][7] = King.new('black', [4,7], 'step')
        array[0][7] = Rook.new('black', [0,7], 'slide')
        array[7][7] = Rook.new('black', [7,7], 'slide')
        Board.new(array)
      end
      it "returns an array with 1st element eq to true if king can castle" do
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
         game.stub(:board) { board }
        expect(game.move_possible([4,0], [6,0]).first).to eq true
      end
    end
    context "regards to castling, left top" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[0][0] = Rook.new('white', [0,0], 'slide')
        array[7][0] = Rook.new('white', [7,0], 'slide')
        array[4][7] = King.new('black', [4,7], 'step')
        array[0][7] = Rook.new('black', [0,7], 'slide')
        array[7][7] = Rook.new('black', [7,7], 'slide')
        Board.new(array)
      end
      it "returns an array with 1st element eq to true if king can castle" do
        game.stub(:current_player) { ginny }
        game.stub(:other_player) { flo }
        game.stub(:board) { board }
        expect(game.move_possible([4,7], [2,7]).first).to eq true
      end
    end
    context "regards to castling, right up" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[0][0] = Rook.new('white', [0,0], 'slide')
        array[7][0] = Rook.new('white', [7,0], 'slide')
        array[4][7] = King.new('black', [4,7], 'step')
        array[0][7] = Rook.new('black', [0,7], 'slide')
        array[7][7] = Rook.new('black', [7,7], 'slide')
        Board.new(array)
      end
      it "returns an array with 1st element eq to true if king can castle" do
        game.stub(:current_player) { ginny }
        game.stub(:other_player) { flo }
         game.stub(:board) { board }
        expect(game.move_possible([4,7], [6,7]).first).to eq true
      end
    end
    context "regards to castling" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[0][0] = Rook.new('white', [0,0], 'slide')
        array[5][0] = Queen.new('white', [5,0], 'slide')
        array[4][7] = King.new('black', [4,7], 'step')
        array[0][7] = Rook.new('black', [0,7], 'slide')
        array[3][7] = Queen.new('black', [3,7], 'slide')
        Board.new(array)
      end
      it "returns an array with 1st element eq to false if king can't castle" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        expect(game.move_possible([4,0], [2,0]).first).to eq false
      end
    end
  end

  describe '#king_check?' do  
    context "if check by a rook" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[4][7] = King.new('black', [4,7], 'step')
        array[0][0] = Rook.new('black', [0,0], 'slide')
        Board.new(array)
      end
      it "returns an array" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        king = board.get_case([4,0])
        expect(game.king_check?(king)).to be_kind_of(Rook)
      end
    end

    context "if check by a knight" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[4][7] = King.new('black', [4,7], 'step')
        array[3][2] = Knight.new('black', [3,2], 'step')
        Board.new(array)
      end
      it "returns an array" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        king = board.get_case([4,0])
        expect(game.king_check?(king)).to be_kind_of(Knight)
      end
    end

    context "if check by a bishop" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[4][7] = King.new('black', [4,7], 'step')
        array[3][1] = Bishop.new('black', [3,1], 'slide')
        Board.new(array)
      end
      it "returns an array" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        king = board.get_case([4,0])
        expect(game.king_check?(king)).to be_kind_of(Bishop)
      end
    end

    context "if check by a queen" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[4][7] = King.new('black', [4,7], 'step')
        array[4][4] = Queen.new('black', [4,4], 'slide')
        Board.new(array)
      end
      it "returns an array" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        king = board.get_case([4,0])
        expect(game.king_check?(king)).to be_kind_of(Queen)
      end
    end

    context "if check by a bishop" do
      let(:board) do
        array = Array.new(8) { Array.new(8) }
        array[4][0] = King.new('white', [4,0], 'step')
        array[4][7] = King.new('black', [4,7], 'step')
        array[3][1] = Pawn.new('black', [3,1], 'step')
        Board.new(array)
      end
      it "returns an array" do
        game.stub(:board) { board }
        game.stub(:current_player) { flo }
        game.stub(:other_player) { ginny }
        king = board.get_case([4,0])
        expect(game.king_check?(king)).to be_kind_of(Pawn)
      end
    end
  end

  describe '#victory' do
    let(:board) do
      array = Array.new(8) { Array.new(8) }
      array[0][1] = Pawn.new('white', [0,1], 'step')
      array[1][1] = Pawn.new('white', [1,1], 'step')
      array[2][1] = Pawn.new('white', [2,1], 'step')
      array[0][0] = King.new('white', [0,0], 'step')
      array[4][7] = King.new('black', [4,7], 'step')
      array[5][0] = Queen.new('black', [5,0], 'slide')
      Board.new(array)
    end
    it 'returns true if there is chekmate' do
      game.stub(:board) { board }
      game.stub(:current_player) { flo }
      game.stub(:other_player) { ginny }
      expect(game.victory?).to be true
    end
  end

  describe '#draw' do
    let(:board) do
      array = Array.new(8) { Array.new(8) }
      array[0][0] = King.new('white', [0,0], 'step')
      array[4][7] = King.new('black', [4,7], 'step')
      Board.new(array)
    end
    it 'returns true if draw' do
      game.stub(:board) { board }
      game.stub(:current_player) { ginny }
      game.stub(:other_player) { flo }
      expect(game.draw?).to be true
    end
  end

  describe '#stalemate' do
    let(:board) do
      array = Array.new(8) { Array.new(8) }
      array[0][0] = King.new('white', [0,0], 'step')
      array[4][7] = King.new('black', [4,7], 'step')
      array[1][7] = Queen.new('black', [1,7], 'slide')
      array[5][1] = Rook.new('black', [5,1], 'slide')
      Board.new(array)
    end
    it 'returns true if stalemate' do
      game.stub(:board) { board }
      game.stub(:current_player) { flo }
      game.stub(:other_player) { ginny }
      expect(game.stalemate?).to be true
    end
  end
end
{% endhighlight %}

And regards to design, it pushes you to keep a sort of independance between methods and classes. And this is something really great. That way, if you have to change a functionality in the future, it won't disturb other parts of the code. I'm really greatful to TDD for having learned me that.

## Level up, checkmate!

This feeling when after looping throught debugging and implementing new functionalities, you start a new game and everything works fine.. First it's surprise! I didn't really believed that my game was actually working fine. I have never done a program that long before.

Then it's joy. You've done it! 

[picture joy]

This second part of The Odin Project wasn't a waste of time. I learned how to use Ruby, but more than that I learned how I should design my code. I learned how to use data structures and algorithms to make it more efficient. I learned how to test my code and the beauty of debugging. And I learned how important it is to keep a core clean, tidy and commented.

Next part, Interaction with the real world: Rails !