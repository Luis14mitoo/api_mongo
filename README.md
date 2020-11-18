# api_mongo
Api con conexion a mongo db, es necesario la instalaciòn de la versiòn de mongoose npm install mongoose@4.13.12; para que funcione.

CRUD MONGO SPRESS

npm init --yes
 npm install body-parser ejs express morgan mongoose@4.13.12

 npm install nodemon -D

 "scripts": {
    "dev": "nodemon src/app.js",
    "start": "node src/app.js"
  },

  *Para uso de plantillas jsx
  *Para conectar a la BBDD mongoose@4.13.12
  *Para entender las peticiones POST body-parser
  *Para ver las peticiones que llegan a mi servidor morgan 
  *JSX lo usamos para poder pasarle variables al HTML


  Carpetas src = libs,model,routes, views


  ====================================================================
  //Archivo para la  coneccion a la base de datos

const mongoose = require('mongoose');

let db;
module.exports = function Connection(){
    if(!db){
        db = mongoose.connect('mongodb://localhost/crud-example', {
            useMongoClient: true
        });
    }

    
    return db;
}

==========================================================================
//Esta archivo configura el schema de la base de datos ya que mongo no tiene esquema()

module.exports = function () {
    var db = require('../libs/db-connection')();
    var Schema = require('mongoose').Schema;


    var Task = Schema({
        title: String, 
        description: String,
        status: Boolean

    });

    return db.model('task', Task);    

}

=========================================================================================
const express = require('express');
const router = express.Router();
const model = require('../model/task')();


//Esta configuracion renderiza al index.jsx
router.get('/', (req, res) => {
    //Aqui se llama a los modulos de la bbdd
    model.find({}, (err, tasks) => {
        if (err) throw err;
        res.render('index', {
            title: 'CRUD',
            tasks: tasks,

        })
    })


});

router.post('/add', (req, res) => {
    let body = req.body;
    body.status = false;

    model.create(body, (err, task) => {
        if (err) throw err;
        res.redirect('/');

    });
});



router.get('/turn/:id', (req, res)=>{
    let id = req.params.id;
    model.findById(id, (err, task) => {
        if(err) throw err;
        task.status= !task.status;
        task.save();

        res.redirect('/');
    });
   
   });
   
   router.get('/delete/:id', (req, res)=>{
    let id = req.params.id;
    model.remove({_id: id}, (err, task) => {
        if(err) throw err;
        

        res.redirect('/');
    });
   
   });


module.exports = router;

==================================================================
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CRUD</title>
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css"
        integrity="sha384-JcKb8q3iqJ61gNV9KGb8thSsNjpSL0n8PARn9HuZOnIxN0hoP+VmmDGMN5t9UJ0Z" crossorigin="anonymous">
</head>

<body>
    <div class="container">
        <form action="/add" method="post">
            <legend>
                Add a Task
            </legend>
            <div class="form-group">
                <input type="text" id="title" class="form-control" name="title">

            </div>
            <div class="form-group">
                <textarea class="form-control" id="description" name="description" placeholder="Add a description"
                    cols="50" rows="1"></textarea>
            </div>

            <button class="btn btn-primary">
                Add
            </button>
           

        </form>


        <form action="">
            <hr>
            <table class="table table-bordered table-hover">
                <thead>
                    <tr>
                        <th>#</th>
                        <th>Title</th>
                        <th>Description</th>
                        <th>Operations</th>
                    </tr>

                </thead>
                <tbody>
                    <% for (var i =0; i < tasks.length; i++) {  %>
                    <tr>
                        <td><%= i  + 1%> </td>
                        <td>
                            <strong>
                                <%= tasks[i].title  %>
                            </strong>
                        </td>
                        <td>
                            <%= tasks[i].description  %>
                        </td>
                        <td>
                            
                            <a 
                            class="<%=tasks[i].status ?  'btn btn-success'
                            : 'btn btn-dark'  %>"
                            href="/turn/<%= tasks[i]._id %>">Done </a>
                            <a
                            id="delete"
                            class="btn btn-danger"
                            href="/delete/<%= tasks[i]._id %>">Delete </a>
                        </td>
                    </tr>
                    <% }    %>
                </tbody>

            </table>


        </form>

    </div>
</body>
<script>
    document.getElementById('title').focus();
    document.getElementById('delete').addEventListener('click',
    function(e){
          let response =  confirm('are you to delete?');
          if(!response){
            e.preventDefault();    
          }
    });
</script>

</html>

  


