# CompteRendu
## Le sujet du stage est d'améliorer le pretty printer de Pharo

# Présentation de Pharo :
+ un language de programmation orienté objet
+ inspiré par Smaltalk
+ pure et élégant
+ dynamiquement typée
+ écrit en lui même.

# Syntaxe :
## éléments syntaxique :
<table>
      <tr>
        <td>commentaire</td>
        <td>"un commentaire"</td>
      </tr>
      <tr>
        <td>caractère</td>
        <td>$a</td>
      </tr>
      <tr>
        <td>chaîne de caractère</td>
        <td>'uneChaine'</td>
      </tr>
      <tr>
        <td>symbole(chaîne unique)</td>
        <td>#unSymbole</td>
      </tr>
      <tr>
        <td>tableau de littéraux</td>
        <td>#(1 2 3 4 5 6 )</td>
      </tr>
      <tr>
        <td>entier</td>
        <td>1</td>
      </tr>
      <tr>
        <td>booléen</td>
        <td>true, false</td>
      </tr>
      <tr>
        <td>indéfinie</td>
        <td>nil</td>
      </tr>
</table>

## constructeurs :

<table>
      <tr>
        <td>declaration de variable temporaires</td>
        <td>| tmp |</td>
      </tr>
      <tr>
        <td>affectation de variables</td>
        <td>tmp:= uneValeur</td>
      </tr>
      <tr>
        <td>séparateur</td>
        <td>message1. message2</td>
      </tr>
      <tr>
        <td>return</td>
        <td>^ uneValeur</td>
      </tr>
      <tr>
        <td>block (fonction anonyme)</td>
        <td>[:x | x+2 ] value: 5</td>
      </tr>
</table>

## les messages dans pharo :
<table>
      <tr>
        <td>message unaire (receveur selecteur)</td>
        <td>3 factorial</td>
      </tr>
      <tr>
        <td>message binaire (receveur selecteur argument)</td>
        <td>1+2</td>
      </tr>
      <tr>
        <td>message à mots clés(receveur clé1:arg1 clé2:arg2)</td>
        <td>2 between: 10 and: 2</td>
      </tr>
</table>

## definition de classe de pharo:

### Exemple de la definition de la classe Point avec deux attributs: x et y

    Object subclass: #Point
    instanceVariableNames: 'x y'
    classVariableNames: ''
    package: 'Graphics'

### definition de methode

    MethodeExample
    "aComment"

    | tmp |
    tmp := 1.
    ^ tmp


# L'AST (Abstract Syntax Trees)
les methodes sont encodés sous forme d'un arbre syntaxique. (composés de différent noeuds)

par exemple:

    | tmp |
    tmp := 1.
    ^ tmp
    
est representé ainsi:

    RBSequenceNode(| tmp| tmp := 1)
        RBVariableNode(tmp)
        RBAssignementNode(tmp := 1)
            RBLiteralValueNode(1)
            RBVariableNode(tmp)

# Le pretty printer
le pretty printer sert a reformater du code suivant une configuration définie

par exemples l'ajout d'espace pour l'assignation d'une variable:

    'tmp:=1' sera reformaté en 'tmp := 1'

Soit une méthode :

        MethodeExample
        "aComment"
        |tmp|
        tmp:=1. ^tmp

le pretty printer reformate cette méthode ainsi:

        MethodeExample
            "aComment"

            | tmp |
            tmp := 1.
            ^ tmp


Il existe des configurations afin de définir les règles de formatage.

# Semaine du 29/04:
J'ai regardé comment le pretty printer est implémenté.

Le RBProgrameNodeVisitor est un visiteur abstrait permettant de visiter les noeuds de l’ast.

Le pretty printer hérite du RBProgrameNodeVisitor.

Il surcharge les méthodes de visite afin de reformater les noeuds visités selon une configuration.

Pour cela il possede un attribut nommé codeStream dans lequel il écrit le résultat.

La configuration est un attribut du pretty printer (contextClass),
elle est une instance de la classe BIPrettyPrinterContext et contient une trentaine de variables.

Par exemple: 
newLinesAfterMethodPattern est un booleen indiquant s'il faut passer une ligne après la signature d'une methode
        

# Semaine du 06/05:
J'ai écrit de tests unitaires sur le formatage de chacun des noeuds syntaxiques en fonction de differentes configurations.

Exemple d'une methode de test verifiant que les éléments d'un assignmentNode soient espacé:    
le test:

    testAssignment   
    | expr source |
    expr := RBParser parseExpression: 'a:=1'.
    configurationSelector := #emptyDefaultConfiguration.
    source := self formatter format: expr.
    self assert: source equals: 'a := 1'
    
la configuration:

    emptyDefaultConfiguration
    "Here we can control explicitely the configuration we want."
    ^ self contextClass new

# Semaine du 13/05:
Suite des tests sur le formatage des noeuds de l'ast.

Cependant les tests présentent un défaut.
    
Les configurations sont toutes crées avec "self contextClass new" qui les initialises avec des valeurs par defaut.

Les tests dépendent donc d'une configuration par défaut (composé d'une trentaine d'attributs).

J'ai donc modifié les configuration en utilisant "self contextClass basicNew".
 
basicNew crée une configuration avec tous ses attributs à nil.
 
En partant de cette configuration vide, j'instancie les attributs nécessaire et laisse les autres à nil.
 
Ce qui me permet de faire les tests en isolant au plus la configuration.

# Semaine du 20/05:
modification et ajout de tests.

correction de configuration pour les tests.

étude des settings:
      les settings sont les attributs de la configuration.
      l'objectif est d'étudier ce que fait chacun et de déterminer s'il y a des settings à renommer, remplacer, ou ajouter.

# Semaine du 27/05:
suite de l'étude des settings.

pair programming sur l'UI du pretty printer:
      Il s'agit d'un menu dans lequel on peut éditer les valeurs de la configuration (les settings)
      et prévisualiser un exemple de formatage avec la configuration.

# Semaine du 3/06:
+ création de méthode d'exemple pour l'UI
	dans l'UI il y a une zone affichant une méthode d'exemple avant et après formatage par le pretty printer.
	la méthode peut être sélectionnée parmis plusieurs méthodes d'exemple.
	j'ai donc fait une méthode pour chaque settings du pretty printer.
      par exemple:

		newLinesAfterTemporariesExample

			| a | a := 1.
     Si le setting newLinesAfterTemporaries vaut true l'UI affichera la méthode avec un retour à la ligne après la déclaration:

			newLinesAfterTemporariesExample

			| a |
			a := 1.
     Si le setting newLinesAfterTemporaries vaut false l'UI affichera la méthode avec un retour à la ligne après la déclaration:
		
			newLinesAfterTemporariesExample

			| a | a := 1.
     ainsi on peut modifier les settings et regarder les changements de formatage.

+ Copie des BIConfigurableFormatter nommée BIEnlumineur et de BIPrettyPrinterContext nommé BIEnlumineurContext.
C'est dans ces classes que j'effectue les modifications.

+ modification des tests pour en faire des tests paramétrés.
les tests prennent en paramètres le formater et la configuration.

+ Début des modifications.
J'ai commencé a effectuer des modifications dans la classe BIEnlumineur afin de corriger des problèmes du pretty printer.

par exemple le code suivant: |a"comment"b| 

était formaté ainsi :| a"comment" b |

est désormais est formaté: | a "comment" b |
