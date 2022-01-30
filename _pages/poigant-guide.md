---
title: "Poignant Guide to Ruby - Anotações João Gilberto"
description: "Caderno de anotações da leitura do livro"
layout: post
tags: ruby rails poignant guide why livro book caderno anotacoes
categories: misc
image:
---

## Exemplos de códigos:

5.times { print "Odelay!" }

exit unless "restaurant".include? "aura"

['toast', 'cheese', 'wine'].each { |food| print fod.capitalize }

## Partes do discurso

phone_a_quail também é uma variável

-1000 também é um inteiro assim como 12_000_000 (Ruby aceita _ para torar o numero mais legível)

12.043e-04 também é um float, já que é uma notação científica

:a :ponce_de_leon são exemplos de symbols, strings mais velozes usadas quando não se precisa imprimir ela na tela.

EmpireStateBuilding = "350 5th Avenue, NYC, NY" (Constantes usam CamelCase)

# Métodos

front_door.open.close (métodos podem usar . e ?)

front_door.is_open?

front_door.paint( 3, :red ) (Método que precisa de argumentos)

front_door.paint( 3, :red).dry( 30 ).close() (métodos encadeados)

## Métodos de classe

Door::new( :oak ) (utilizam os ::)

## Variáveis globais

$x $CHunKY_bACOn (começam com o $)

## Variáveis de instância e de classe

@width @x são variáveis de instancia (caraceterísticas de um objeto único)

@@x @@time são variáveis da classe como um todo

# Blocos

2.times {
  print "Testando 123, teste 123, teste"
}

Todo código entre { } é um bloco, um conjunto de instruções.

loop do
  print "Much better."
  print "Ah. More space!"
  print "My back was killin' me in those crab pincers."
end

Quando o bloco passa de uma linha, melhor usar do e end.

## Argmentos dos blocos

|x|, |x,y| e |up, down, all_around| são exemplos de variáveis utilizadas no começo do bloco.

{ |x,y| x + y }

## Ranges

(1...3) aqui não inclui o 3
('a'..'z') lembrar que mais é menos, aqui inclui o z

## Arrays

['coat', 'mittens', 'snowboard'] lembrar que os itens são guardados numa ordem específica

## Hashes

{'Name' => 'Peter', 'profession' => 'lion tamer'}

## Métodos

Métodos como mensagens, atenção as respostas que eles dão.

## Nil

Representa o vazio, sem valor. É diferente de zero.
Tudo em Ruby tem uma carga positiva

## Uso invertido do if e unless

print "We're using plastic 'cause we don't have glass." if plastic_cup unless glass_cup

This trick is a gorgeous way of expressing, Do this only if a* is true and *b isn’t true.

## == como método

approaching_guy.== ( true )

if nil.==( true )
  print "This will never see realization."
end

at_hotel = true
email = if at_hotel
          "why@hotelambrose.com"
        else
          "why@drnhowardcham.com"
        end

## << como método

email = if at_hotel
          address = "why"
          address << "@hotelambrose"
          address << ".com"
        end

## Variáveis

Uma variável pode ser true ou false. Como em at_hotel = true

Nil pode ser uma resposta capturável e útil. Exemplo:

print( if at_hotel.nil?
         "No clue if he's in the hotel."
       elsif at_hotel == true
         "Definitely in."
       elsif at_hotel == false
         "He's out."
       else
         "The system is on the freee-itz."
       end )

## GSUB

gsub em Ruby é global substitution, utilizando com ! troca as palavras na origem. Torna-se assim um método destrutivo.

CODE_WORDS.each do |real, code|
  idea.gsub!( real, code )
end

## Strip

Ele retira os espaços em branco e afins do começo e fim das strings.

print "File encoded.  Please enter a name for this idea: "
idea_name = gets.strip
File::open( 'idea-' + idea_name + '.txt', 'w' ) do |f|
  f << idea
end

File::open cria um novo arquivo. Os métodos read, rename, delete estão na classe File do Kernel.

No exemplo acima, atenção para o w, escreve em um novo arquivo. Com r seria o modo ler, já a o modo escrever no fim do arquivo.
No fim acrescenta a ideia no arquivo com << (um método que existe em File e também em strings).

## Dir

Dir['idea-*.txt'].each do |file_name|
  idea = File.read( file_name )
  CODE_WORDS.each do |real, code|
    idea.gsub!( code, real )
  end
  puts idea
end*

O Dir::[] pesquisa no diretório, já o * é um coringa que permite que ele procure algo que com 'ideia-*.txt'. *O Dir[] forma um array de strings que são os nomes dos arquivos de ideias achados na pasta.

p Dir['ideia-*.txt] 

*Diferente de print, p permite mostrar qualquer coisa, não apenas strings.

## Array de Hashs

kitty_toys =
  [:shape => 'sock', :fabric => 'cashmere'] +
  [:shape => 'mouse', :fabric => 'calico'] +
  [:shape => 'eggroll', :fabric => 'chenille']
kitty_toys.sort_by { |toy| toy[:fabric] }

Reorganiza o array de hashs pelo fabric

kitty_toys.sort_by { |toy| toy[:shape] }.each do |toy|
  puts "Blixy has a #{ toy[:shape] } made of #{ toy[:fabric] }"
end

Reorganiza e coloca a frase usando pedaços do dicionário

## Next e break if

non_eggroll = 0
kitty_toys.each do |toy|
  next if toy[:shape] == 'eggroll'
  non_eggroll = non_eggroll + 1
end

No caso, ele está contando os brinquedos, mas pula se for o eggroll

kitty_toys.each do |toy|
  break if toy[:fabric] == 'chenille'
  p toy
end

Ai ele vai imprimindo os brinquedos (usando p porque são hashs) e para o laço se achar o chenille

# ===

Não é tão restritivo quando o ==

if 1894 === year
  "Born."
elsif (1895..1913) === year
  "Childhood in Lousville, Winston Co., Mississippi."
else
  "No information about this year."
end

# Scopes

$ global
@ instancia: geralmente usado para guardar pedaços de informação sobre o objeto da classe. Um atributo da classe
@@ classe
   local

Variáveis de instancia são acessíveis dentro de um objeto, já as de classe, dentro da classe em questão.

## Método .new

Exclusivo da classe, cria o objeto e chama automaticamente o método initialize

## Tudo é um objeto

Tudo em Ruby é um objeto!

## def initialize

É ativada automaticamente quando se cria um objeto.

## Utilizar o raise para criar mensagens de erro

def wipe_mutterings_from( sentence )
  unless sentence.respond_to? :include?
    raise ArgumentError,
      "cannot wipe mutterings from a #{ sentence.class }"
  end
  sentence = sentence.dup
  while sentence.include? '('
    open = sentence.index( '(' )
    close = sentence.index( ')', open )
    sentence[open..close] = '' if close
  end
  sentence
end

respoind_to? testa se o objeto tem o método :include? (usa o : para detonar que é do Ruby e acelerar a pesquisa - ele reconhece symbols mais rápido que strings)

O método dub faz uma cópia de qualquer objeto (exceto números e symbols)

# Reescrevendo para evitar ficar preso no loop de uma frase sem fechar o parêntesis

def wipe_mutterings_from( sentence )
  unless sentence.respond_to? :gsub
    raise ArgumentError,
      "cannot wipe mutterings from a #{ sentence.class }"
  end
  sentence.gsub( /\([-\w]+\)/, '' )
end

# Dicas sobre métodos

1. As pessoas vão passar os tipos errados nos métodos, crie os raise adequados para isso.
2. A etiqueta diz que não é correto mudar objetos através de métodos, crie cópias usando dup, gsub e outras saídas.
3. Os colchetes [ ] podem ser usados do lado esquerdo do = para pesquisar/alterar parte dos objetos.
4. Evitar utilizar until e while.

class String

  # The parts of my daughter's organ
  # instructor's name.
  @@syllables = [ # uma variável de classe, atributo.
    { 'Paij' => 'Personal',
      'Gonk' => 'Business',
      'Blon' => 'Slave',
      'Stro' => 'Master',
      'Wert' => 'Father',
      'Onnn' => 'Mother' },
    { 'ree'  => 'AM',
      'plo'  => 'PM' }
  ]

  # A method to determine what a
  # certain name of his means.
  def name_significance
    parts = self.split( '-' ) # self para fazer a quebra de si mesmo.
    syllables = @@syllables.dup
    signif = parts.collect do |p| 
      syllables.shift[p]
    end
    signif.join( ' ' )
  end

end 

Collect é um each que no fim pega a resposta do bloco e coloca em uma nova array. Usado para criar uma nova array com elementos de uma array existente.
Shift remove o primeiro elemento de self e retorna ele ou então nil se for uma array vazia.

catsandtips = [0.12, 0.63, 0.09].collect { |catcost| catcost + ( catcost * 0.20 ) }

# Usando o self

class String
  def dash_split
    self.split( '-' )
  end
end

"Gonk-plo".dash_split
#=> ['Gonk', 'plo']

# Classe
- Toda classe em ruby herda de Object
- Herança é útil para sobrescrever comportamentos de uma classe

irb> ArrayMine.superclass
  => Array

# Module

- Módulos são "tetos" para abrir um conjunto de classes, contantes, variáveis etc.
- Mas não é uma classe em si, não pode dar um "Module.new"

# Ticket de loteria

class LotteryTicket

  NUMERIC_RANGE = 1..25

  attr_reader :picks, :purchased

  def initialize( *picks )
    if picks.length != 3
      raise ArgumentError, "three numbers must be picked"
    elsif picks.uniq.length != 3
      raise ArgumentError, "the three picks must be different numbers"
    elsif picks.detect { |p| not NUMERIC_RANGE === p }
      raise ArgumentError, "the three picks must be numbers between 1 and 25"
    end
    @picks = picks
    @purchased = Time.now
  end

end

- O * em *picks indica que quaisquer argumentos podem ser passados.
- O attr_reader é o mesmo que

class LotteryTicket
  def picks; @picks; end
  def purchased; @purchased; end
end

Isso permite que eles sejam usados fora da classe.

class LotteryTicket
  def self.new_random
    new( rand( 25 ) + 1, rand( 25 ) + 1, rand( 25 ) + 1 )
  rescue ArgumentError
    retry
  end
end

- Criando o ticket aleatório, tratando o erro em caso de repetição de números e reinicializando.

# Module

Mais simples que uma classe, serve para guardar e agrupar facilmente métodos. Não dá para criar objetos de um módulo.

# Mixin

*Um módulo incluido numa classe é um mixin dessa classe. Atenção que é necessário um método read para que se tenha acesso aos métodos.

require 'mindreader'
class MindReader
  include WishScanner
ends

# Open-Uri

require 'open-uri'
open( "http://preeventualist.org/lost/searchfound?q=truck" ) do |truck|
  truck.each_line do |line|
    puts line if line['pickup']
  end
end

O método shift pega o primeiro item e continua.

irb> a = ['first, birth.', 'then, a life of flickering images.', 'and, finally, the end.']
irb> yield_thrice { puts a.shift }

# Dois modos de escrever o bloco

# The brief style of attaching a block to a method.
# Here the block is surrounded with curly braces.
open( "idea.txt" ) { |f| f.read }

# The verbose style of attaching a block to a method.
# Here the block is surrounded with `do' and `end'
open( "idea.txt" ) do |f|
  f.read
end

# Yield em um método

Usando para retirar o arquivo 1 e 2 para ser usando abaixo. Ele sai do método para voltar ao resto do código.

# The method opens two files and slides the resulting IO objects down the
# chute to an attached block.
def double_open filename1, filename2
  open( filename1 ) do |f1|
    open( filename2 ) do |f2|
      yield f1, f2
    end
  end
end

# Prints the files out side-by-side.
double_open( "idea1.txt", "idea2.txt" ) do |f1, f2|
  puts f1.readline + " | " + f2.readline
end

# Eval
Executa o código armazenado em uma string.

print "What monster class have you come to battle? "
monster_class = gets
eval( "monster = " + monster_class + ".new" )
p monster

Já instance_eval e class_eval fazem isso com um bloco, não com uma string.

# The instance_eval method runs code as if it were run inside an
# object's instance method.
irb> drgn = Dragon.new
irb> drgn.instance_eval do
irb>   @name = "Tobias" 
irb> end

irb> drgn.instance_variable_get( "@name" )
  => "Tobias" 

# The class_eval method runs code is if inside a class definition.
irb> Dragon.class_eval do
irb>   def name; @name; end
irb> end

irb> drgn.name
  => "Tobias"

# Inspect
Método de todos objetos em Ruby, retorna uma "inspeção do que é o objeto". Isso é a base do irb.

"Teste".inspect

# method_missing
Um método que intercepta chamadas de métodos que não existem.

def self.method_missing(method, *args, &block)
end

method: é o método que falta ao objeto, geralmente é convertido em string antes de ser usado. É um parametro requerido.

*args: uma esponja para pegar argumentos passados a mais, opcional.

&blocks: inclui bloco chamados com o método. Mais um parametro opcional.

# %s
Une uma string e um array para criar um novo string

# %d
Idem com números

# %
União mais flexível

irb> "Seats are taken by %s and %s." % ['a frog', 'a frog with teeth']
  => "Seats are taken by a frog and a frog with teeth.

irb> frogs = [44, 162.30]
irb> stats = "Frogs have filled %d seats and paid %f blue crystals." 
irb> stats % frogs
  => "Frogs have filled 44 seats and paid 162.30 blue crystals."

irb> frogs = ['44', '162.30']
irb> stats % frogs
  => "Frogs have filled 44 seats and paid 162.30 blue crystals."

# De outros modos
irb> "Here is your 1098 statement for the year, %s." % ['teeth frog']
  => "Here is your 1098 statement for the year, teeth frog.

irb> format "Frogs are piled %d deep and travel at %d mph.", [5, 56]
  => "Frogs are piled 5 deep and travel at 56 mph."i

irb> "This bus has %1$d more stops before %2$d o'clock.  That's %1$d more stops." % [16, 8]
  => "This bus has 16 more stops before 8 o'clock.  That's 16 more stops."

# Give one left-justified item 30 characters of width
irb> "At the front of the bus: %-30s." % ['frogs']
  => "At the front of the bus: frogs  

irb> fellows = ['Blix', 'Fox Tall', 'Fox Small']
irb> puts "Let us follow #{ fellows.join ' and ' } on their journey." 
  => "Let us follow Blix and Fox Tall and Fox Small on their journey."

irb> blix_went = :north
irb> puts "Blix didn't speak, he ducked off to the #{ blix_went } through #{
            if blix_went == :north
              'a poorly laid avenue behind the paint store'
            elsif blix_went == :south
              'the circuitry of apartment buildings'
            else
              '... well, who knows where he went.'
            end }.  But before we follow them..." 

# Em resumo
%s string 
%d inteiros
%f float
%p placehouder, vai chamar o inspect

# $
Ele comporta todos os diretórios que o Ruby pesquisa quando tenta fazer o load de um arquivo via require

irb> $:
  => ["/usr/lib/ruby/site_ruby/1.8", "/usr/lib/ruby/site_ruby/1.8/i686-linux",
      "/usr/lib/ruby/site_ruby", "/usr/lib/ruby/1.8",
      "/usr/lib/ruby/1.8/i686-linux"]

$0 - o nome do arquivo que está rodando o programa
$* - *Uma variável que contem todos argumentos passados ao programa
$! exceção que no momento corrente foi levantada
$@ os dados de ontem a exceção foi chamada.

# $/ /n $

$/ - redefine o que é o separador de linhas
"\n" - normalmente é o que representa um ENTER, quebra a linha na String. Controla métodos como each_line e readlines

irb> "Jeff,Jerry,Jill\nMichael,Mary,Myrtle".each_line { |names| p names }
  => "Jeff,Jerry,Jill\n" 
  => "Michael,Mary,Myrtle" 

irb> $/ = ','
irb> "Jeff,Jerry,Jill\nMichael,Mary,Myrtle".each_line { |names| p names }
  => "Jeff," 
  => "Jerry," 
  => "Jill\nMichael," 
  => "Mary," 
  => "Myrtle" 

$ - variável que é guarda a união de linhas, o join. A vírgula é o mais comum.

irb> ['candle', 'soup', 'mackarel'].join
  => "candlesoupmackarel" 
irb> $, = ' * '; ['candle', 'soup', 'mackarel'].join
  => "candle * soup * mackarel" 

# Normalmente você não precisa dessa variável global já que:
irb> ['candle', 'soup', 'mackarel'].join ' # '
  => "candle # soup # mackarel

$; - guarda a variável da separação de linhas. Geralmente é vazia.

irb> "candle  soup\nmackarel".split
  => ["candle", "soup", "mackarel"]
irb> $; = 'a'; "candle  soup\nmackarel".split
  => ["c", "ndle  soup\nm", "ck", "rel"]

# Mas, assim como no caso anterior, dá para fazer:
irb> "candle # soup # mackarel".split ' # '
  => ['candle', 'soup', 'mackarel'

# %w
Cria uma array de strings.

irb> bats = %w{The Winged Scroll Carriers}
  => ['The', 'Winged', 'Scroll', 'Carriers']

# %x
Roda programas externos

irb> %x{ruby --help}
  => "Usage: ruby [switches] [--] [programfile] [arguments] ..."

# %Q ou %
- Cria uma string com " " mas é útil para fazer rodar várias linhas de uma vez.

m = "bats!"
eval %(
  def #{ m }
    puts "{" * 100
  end
)

# clone
Cria uma cópia do objeto, não apenas dá um novo rótulo para ele (como seria nomear com uma nova variável). Com um clone é possível modificar a cópia sem mexer no original. O clone também leva os estados dos objetos, já o dup não.

vaca = [:uma, :duas]
cows = vaca
vaquinhas = vaca.clone

cows << 'tres'

# Então

vaca = [:uma, :duas, tres]
cows = [:uma, :duas, tres]
vaquinhas = [:uma, :duas]

# dup
Outra forma de copiar, mas não leva o estado dos objetos.

irb> o = Object.new
irb> class << o
irb>   def nevermore; :nevermore; end
irb> end

irb> o.clone.nevermore
  => :nevermore
irb> o.dup.nevermore
# NoMethodError: undefined method `nevermore' for #<Object:0xb7d4a484>
#         from (irb):7

# Obs
Diversos métodos fazem cópias para realizar seu trabalho: colletec, gsub e format.

# Regular Expressions

loop do
  print "Enter your password: "
  password = gets
  if password.match( /^\w{8,15}$/ )
    break
  else
    puts "** Bad password! Must be 8 to 15 characters!"
  end
end

No caso, aceita letras, números e underlines, entre 8 e 15 caracteres.

irb> "good_password".match( /^\w{8,15}$/ )
  => #<MatchData:0xb7d54218>
irb> "this_bad_password_too_long".match( /^\w{8,15}$/ )
  => nil

# Alguns atalhos usando Regex

require 'preeventualist'
PreEventualist.searchfound( 'truck' ) do |page|
  page.each_line do |line|
    puts line if line.match( /truck/i )
  end
end

Agora deixa de ser case sensitive, pesquisa truck em todas suas possibilidades de minúsculas e maiúsculas.

puts line if line.match( /T-\d000/ )

Todos os modelos T- com 3 números depois T-1000, T-2000 etc.

# Classes de Caracteres
\d - digitos [0-9]
\w - caracteres incluindo underline, pode ser escrito como [A-Za-z0-9_]
\s - espaços, tabs e outros espaços brancos. a.k.a. [ \t\r\n]
\D - Tudo menos dígitos 
\W  - qualquer caractere que não seja uma palavras 
\S - qualquer caractere que não seja um espaço em branco 
. - pega qualquer coisa

/\d\d\d/ - Buscando 3 números, regex começa em e termina em /

/\d{3}/ - A mesma pesquisa

# Quantificadores

{n} - exatamente n vezes
{n,} -  n vezes ou mais.

/[a-z]{3,}/i 

No exemplo, três ou mais letras em sequencia. Seja maíscula ou minúscula.

{n,n2} - pelo menos  n_ vezes mas não mais que _n2 vezes 

/[\d,]{3,9}/

Exemplo: De três a 9 caracteres que são dígitos ou virgulas.

* é um atalho para  {0,}. 
Para pegar dois pontos seguido por o ou mais caracteres. /:\w*/

+ atalho para {1,0}
Para pegar um ou mais + ou - , use /[-+]+/

? um atalho para {0,1} 
Para pegar 3 números seguidos de um período opcional use /\d{3}[.]?/

# match, =~ e $&

irb> "Call 909-375-4434" =~ /\d{3}-\d{3}-\d{4}/
  => 5
irb> "The number is (909) 375-4434" =~ /[(]\d{3}[)]\s*\d{3}-\d{4}/
  => 14

O =~ faz o comparativo e match, ele retorna quantos caracteres há antes do match atender ao que você passou.

O match faz a comparação se está dentro dos parâmetros. Salva um objeto MatchData

Já o $& é uma variável global que guarda o que você passou usando o =~

## Exemplos

### =~ e $&

irb> "The number is (909) 375-4434" =~ /[(]\d{3}[)]\s*\d{3}-\d{4}/
  => 14
irb> $&
  => "(909) 375-4434"  

### MatchData object

irb> phone = /[(]\d{3}[)]\s*\d{3}-\d{4}/.match("The number is (909) 375-4434")     
  => #<MatchData:0xb7d5168>
irb> phone.to_s
  => "(909) 375-4434"

# Procurar e substituir
Um dos usos mais comuns para regex

irb> song = "I swiped your cat / And I stole your cathodes" 
irb> song.gsub 'cat', 'banjo'
  => "I swiped your banjo / And I stole your banjohodes" 

irb> song.gsub /\bcat\b/, 'banjo'
  => "I swiped your banjo / And I stole your cathodes"

No caso, \b no começo e fim significa que precisa ser a palavra completa igual. Há variações no uso do \b só no começo ou no fim.			
