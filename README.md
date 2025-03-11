# Système de Gestion de Tâches avec GraphQL 🚀

## OBJECTIF(S) 🎯
- Découvrir et implémenter **GraphQL** avec **Node.js** et **Express.js** ⚡
- Créer une API pour gérer les tâches avec des **requêtes** et **mutations** GraphQL 🏗️

## OUTILS UTILISÉS 🛠️
- **Node.js** ⚙️
- **Express.js** 🌟
- **GraphQL** 🔍
- **Apollo Server** 📡

### Qu'est-ce que **GraphQL** ? 🤔
**GraphQL** est un langage de requête pour les API, permettant aux clients de récupérer uniquement les données dont ils ont besoin, ce qui le rend plus efficace que les API REST traditionnelles. Il permet également une meilleure gestion des mutations et des requêtes complexes.

---

## Étapes de Création 📝

### Étape 1 : Initialisation du Projet 🏗️
1. Créez un dossier pour votre projet et accédez-y :
    ```bash
    mkdir tp-graphql
    cd tp-graphql
    ```
2. Initialisez un projet **Node.js** avec :
    ```bash
    npm init -y
    ```
3. Installez les dépendances nécessaires :
    ```bash
    npm install express @apollo/server body-parser @graphql-tools/schema graphql
    ```

### Étape 2 : Création du Schéma GraphQL 📜
1. Créez un fichier `taskSchema.gql` avec le contenu suivant :

    ```graphql
    type Task {
        id: ID!
        title: String!
        description: String!
        completed: Boolean!
        duration: Int
    }

    type Query {
        task(id: ID!): Task
        tasks: [Task]
    }

    type Mutation {
        addTask(title: String!, description: String!, completed: Boolean!, duration: Int): Task
        completeTask(id: ID!): Task
        changeDescription(id: ID!, description: String!): Task
        deleteTask(id: ID!): Task
    }
    ```

2. Créez un fichier `taskSchema.js` qui importe le schéma et le transforme en schéma GraphQL :

    ```javascript
    const fs = require('fs');
    const path = require('path');
    const { buildSchema } = require('graphql');
    const { promisify } = require('util');
    const readFileAsync = promisify(fs.readFile);

    async function getTaskSchema() {
        const schemaPath = path.join(__dirname, 'taskSchema.gql');
        try {
            const schemaString = await readFileAsync(schemaPath, { encoding: 'utf8' });
            return buildSchema(schemaString);
        } catch (error) {
            console.error("Error reading the schema file:", error);
            throw error;
        }
    }
    module.exports = getTaskSchema();
    ```

### Étape 3 : Définir les Résolveurs 🛠️
1. Créez un fichier `taskResolver.js` pour définir les résolveurs pour les tâches :

    ```javascript
    let tasks = [
        {
            id: '1',
            title: 'Développement Front-end pour Site E-commerce',
            description: 'Créer une interface utilisateur réactive en utilisant React et Redux.',
            completed: false,
            duration: 120
        },
        {
            id: '2',
            title: 'Développement Back-end pour Authentification',
            description: 'Implémenter un système d\'authentification avec Node.js, Express, et Passport.js.',
            completed: false,
            duration: 90
        }
    ];

    const taskResolver = {
        Query: {
            task: (_, { id }) => tasks.find(task => task.id === id),
            tasks: () => tasks,
        },
        Mutation: {
            addTask: (_, { title, description, completed, duration }) => {
                const task = {
                    id: String(tasks.length + 1),
                    title,
                    description,
                    completed,
                    duration
                };
                tasks.push(task);
                return task;
            },
            completeTask: (_, { id }) => {
                const taskIndex = tasks.findIndex(task => task.id === id);
                if (taskIndex !== -1) {
                    tasks[taskIndex].completed = true;
                    return tasks[taskIndex];
                }
                return null;
            },
            changeDescription: (_, { id, description }) => {
                const task = tasks.find(task => task.id === id);
                if (task) {
                    task.description = description;
                    return task;
                }
                return null;
            },
            deleteTask: (_, { id }) => {
                const taskIndex = tasks.findIndex(task => task.id === id);
                if (taskIndex !== -1) {
                    const [deletedTask] = tasks.splice(taskIndex, 1);
                    return deletedTask;
                }
                return null;
            }
        }
    };

    module.exports = taskResolver;
    ```

### Étape 4 : Configuration du Serveur Express avec GraphQL 🌍
1. Créez un fichier `index.js` et configurez le serveur **Express** avec **Apollo Server** :

    ```javascript
    const express = require('express');
    const { ApolloServer } = require('@apollo/server');
    const { expressMiddleware } = require('@apollo/server/express4');
    const { json } = require('body-parser');
    const { addResolversToSchema } = require('@graphql-tools/schema');
    const taskSchemaPromise = require('./taskSchema');
    const taskResolver = require('./taskResolver');

    const app = express();

    async function setupServer() {
        try {
            const taskSchema = await taskSchemaPromise;
            const schemaWithResolvers = addResolversToSchema({
                schema: taskSchema,
                resolvers: taskResolver,
            });

            const server = new ApolloServer({
                schema: schemaWithResolvers,
            });

            await server.start();
            app.use('/graphql', json(), expressMiddleware(server));

            const PORT = process.env.PORT || 5000;
            app.listen(PORT, () => {
                console.log(`🚀 Serveur GraphQL en cours d'exécution sur http://localhost:${PORT}/graphql`);
            });
        } catch (error) {
            console.error('Erreur lors du démarrage du serveur Apollo:', error);
        }
    }

    setupServer();
    ```

### Étape 5 : Tester l'API avec GraphiQL 🧪
1. Ouvrez l'interface de test **GraphiQL** dans votre navigateur à l'adresse [http://localhost:5000/graphql](http://localhost:5000/graphql).
2. Vous pouvez tester les requêtes et mutations suivantes :

#### Récupérer toutes les tâches :
```graphql
{
  tasks {
    id
    title
    description
    completed
  }
}
```

#### Ajouter une nouvelle tâche :
```graphql
mutation {
  addTask(title: "Nouvelle tâche", description: "Description de la tâche", completed: false, duration: 60) {
    id
    title
    description
    completed
    duration
  }
}
```

#### Marquer une tâche comme terminée :
```graphql
mutation {
  completeTask(id: "1") {
    id
    completed
  }
}
```

#### Changer la description d'une tâche :
```graphql
mutation {
  changeDescription(id: "1", description: "Nouvelle description mise à jour") {
    id
    title
    description
  }
}
```

#### Supprimer une tâche :
```graphql
mutation {
  deleteTask(id: "1") {
    id
    title
  }
}
```
