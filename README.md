# CompteRendu
#Présentation de Pharo :

            un language de programmation orienté objet
            inspiré par Smaltalk
            pure et élégant
            Dynamiquement typée
            écrit en lui même.
#Syntaxe :

    éléments syntaxique :
        commentaire             "un commentaire"
        caractère               $a
        chaîne de caractère     'uneChaine'
        symbole(chaîne unique)  #unSymbole
        tableau de littéraux    #(1 2 3 4 5 6 )
        entier                  1
        booléen                 true, false
        indéfinie               nil

    constructeurs :
        declaration de variable temporaires     | tmp |
        affectation de variables                tmp:= uneValeur
        séparateur                              message1. Message2
        return                                  ^ uneValeur
        block (fonction anonyme)                [:x | x+2 ] value: 5


    les messages dans pharo :

        message unaire (receveur selecteur)
            exemple: 3 factorial

        message binaire : receveur selecteur argument
            exemple: 1+2

        message à mots clés : receveur clé1:arg1 clé2:arg2
            exemple: 2 between : 10 and: 20

    definition de classe de pharo:

        Exemple de la definition de la classe Point avec deux attributs: x et y

            Object subclass: #Point
            instanceVariableNames: 'x y'
            classVariableNames: ''
            package: 'Graphics'

    definition de methode:

        MethodeExample
        "aComment"

        | tmp |
        tmp := 1.
        ^ tmp


#L'AST (Abstract Syntax Trees)
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





#Le pretty printer

    le pretty printer sert a reformater du code suivant une configuration définie

    exemples:
        ajout d'espace pour l'assignation d'une variable:
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
Le sujet du stage est d'améliorer le pretty printer

#Semaine du 29/04:

    J'ai regardé comment le pretty printer est implémenté.
    Le RBProgrameNodeVisitor est un visiteur abstrait permettant de visiter les noeuds de l’ast.

    Le pretty printer hérite du RBProgrameNodeVisitor.
    Il surcharge les méthodes de visite afin de reformater les noeuds visités selon une configuration.
    Pour cela il possede un attribut nommé codeStream dans lequel il écrit le résultat.
    La configuration est un attribut du pretty printer (contextClass),
    elle est une instance de la classe BIPrettyPrinterContext et contient une trentaine de variables.
    par exemple: 

        newLinesAfterMethodPattern: un booleen indiquant s'il faut passer une ligne après la signature d'une methode
        

#Semaine du 06/05:
    tests unitaires sur le formatage des différents noeuds syntaxiques en fonction d'une configuration donné.

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

#Semaine du 13/05:
    suite des tests sur le formatage des noeuds de l'ast.
    Cependant les tests présentent un défaut.
    
    les configurations sont toutes crées avec "self contextClass new" qui les initialises avec des valeurs par defaut.
    les tests dépendent donc d'une configuration par défaut (composé d'une trentaine d'attributs).
    
    J'ai donc modifié les configuration en utilisant "self contextClass basicNew"
    basic new crée une configuration avec tout ces attributs à nil.
    en partant de cette configuration vide, j'instancie les attributs nécessaire et laisse les autres à nil.
    ce qui me permet de faire des tests en isolant au plus la configuration.
