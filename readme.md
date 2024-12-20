# Урок 4. Создание REST API с Express
Для того, чтобы пользователи хранились постоянно, а не только, когда запущен сервер, необходимо реализовать хранение массива в файле.

Подсказки:
— В обработчиках получения данных по пользователю нужно читать файл \
— В обработчиках создания, обновления и удаления нужно файл читать, чтобы убедиться, что пользователь существует, а затем сохранить в файл, когда внесены изменения \
— Не забывайте про JSON.parse() и JSON.stringify() - эти функции помогут вам переводить объект в строку и наоборот.

## index.js

```
const express = require('express');
const joi = require('joi');

const fs = require('fs');
const path = require('path');

const app = express();
const port = 3000;

// Пример объекта user

// user = {
//     id: uniqueID,
//     firstname: "Max",
//     secondname: "Powell",
//     age: 39,
//     city: "Birobijan"
// }

function saveJSON(someData, saveTo) {
    fs.writeFileSync(saveTo, JSON.stringify(someData, null, 2));
}

function readJSON(readFrom) {
    const data = fs.readFileSync(readFrom);
    if (data) {
        return JSON.parse(fs.readFileSync(readFrom));
    } else {
        return [];
    }
}

const usersPath = path.join(__dirname, "users.json");

let users = [];
let uniqueID = 0;

// Если файл json существует читаем в users, иначе пишем в новый файл json массив объектов users (в начальном варианте пустой)
if (fs.existsSync(usersPath)) {
    users = readJSON(usersPath);
    // Если JSON не пустой присваиваем макисмальное значение идентификатору uniqueID
    if (users) {
        uniqueID = users[users.length - 1].id;
    }
} else {
    saveJSON(users, usersPath);
}

const userSchema = joi.object({
    firstname: joi.string().min(2).required(),
    secondname: joi.string().min(3).required(),
    age: joi.number().min(0).max(150).required(),
    city: joi.string().min(2)
});

app.use(express.json());

app.get("/users", (req, res) => {
    users = readJSON(usersPath);
    res.send({ users });
});

app.get("/users/:id", (req, res) => {
    users = readJSON(usersPath);

    const user = users.find(user => user.id === Number(req.params.id));
    if (user) {
        res.send({ user });
    } else {
        res.status(404).send({ user: null });
    }
});

app.post("/users", (req, res) => {
    const result = userSchema.validate(req.body);
    if (result.error) {
        return res.status(400).send({error: result.error.details});
    }

    users = readJSON(usersPath);

    const user = users.find((user) => {return user.firstname === req.body.firstname && 
                                    user.secondname === req.body.secondname &&
                                    user.age === Number(req.body.age) &&
                                    user.city === req.body.city
    });

    if (!user) {
        uniqueID += 1;

        users.push({
            id: uniqueID,
            ...req.body
        });

        saveJSON(users, usersPath);

        res.send({id: uniqueID});
    } else {
        res.status(404).send({user : null});
    }
});

app.put("/users/:id", (req, res) => {
    const result = userSchema.validate(req.body);
    if (result.error) {
        return res.status(400).send({error: result.error.details});
    }

    users = readJSON(usersPath);

    const user = users.find(user => user.id === Number(req.params.id));
    if (user) {
        user.firstname = req.body.firstname;
        user.secondname = req.body.secondname;
        user.age = req.body.age;
        user.city = req.body.city;

        saveJSON(users, usersPath);

        res.send({user});
    } else {
        res.status(404).send({user : null});
    }
});

app.delete("/users/:id", (req, res) => {
    users = readJSON(usersPath);

    const user = users.find(user => user.id === Number(req.params.id));
    if (user) {
        const userIndex = users.indexOf(user);
        users.splice(userIndex, 1);

        saveJSON(users, usersPath);

        res.send({user});
    } else {
        res.status(404).send({user : null});
    }
});

app.listen(port, () => {
    console.log(`Сервер запущен на ${port} порту`);
});
```