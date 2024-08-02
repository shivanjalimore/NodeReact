//---------------------------router.js --------------------------------<br>
const express=require("express")
const myrouter=express.Router();
const connection=require("../db/dbconnect")
//to login page
myrouter.get('/login', function(req, res){
    res.render('login');
});
//check login credentials
myrouter.post('/login', function(req, res) {
    connection.query('SELECT * FROM user WHERE email = ? AND password = ?',[req.body.email, req.body.password], 
        function(err, data){
        if (!err) {
            res.redirect('/courses');
        } else {
            res.send('Invalid email or password');
        }
    });
});
//get all courses
myrouter.get("/courses",function(req,resp){
    connection.query("select * from course",function(err,data){
        if(err){
            resp.status(500).send("Error Occured!!")
        }
        else{
            resp.status(200).render("displaycourse",{coursdata:data})
        }
    })
})
//add course
myrouter.post("/insertcourse",function(req,resp){
    connection.query("insert into course values(?,?,?,?)",[req.body.cid,req.body.cname,req.body.fees,req.body.duration],function(err,result){
        if(err){
            resp.status(500).send("Data not inserted!!")
        }
        else{
                resp.redirect("/courses")
            }
        })
    })
//empty form to add data
myrouter.get("/addcourseform",function(req,resp){
    resp.render("addcourse")
})
//logout
myrouter.get("/logout",function(req,res){
    res.render("login")
})
//delete course
myrouter.get("/deletecourse/:id",function(req,resp){
    connection.query("delete from course where cid=?",[req.params.id],function(err,result){
        if(err){
            resp.status(500).send("Data not found!!")
        }
        else{
                resp.redirect("/courses")
        }
    })
})
//update firstly take id
myrouter.get("/editcourse/:id",function(req,resp){
    connection.query("select * from course where cid=?",[req.params.id],function(err,data){
        if(err){
            resp.status(500).send("Data not found!!")
        }
        else{
            resp.render("updatecourse",{editdata:data[0]})
        }
    })
})
//update actual data
myrouter.post("/updatecourse",function(req,resp){
    connection.query("update course set cname=?,fees=?,duration=? where cid=?",[req.body.cname,req.body.fees,req.body.duration,req.body.cid],function(err,result){
        if(err){
            resp.status(500).send("Data not updated!!")
        }
        else{
            resp.redirect("/courses")
        }
    })
})
module.exports=myrouter;

//------------------------- app.js -------------------------------<br>
const express = require("express")
const app = express()
const path = require("path")
const routes = require("./routes/router")
const bodyParser = require('body-parser')
app.set('views',path.join(__dirname,"views"))
app.set("view engine","ejs")
app.use(bodyParser.urlencoded({extended:false}))
app.use("/",routes)
app.listen(3003,function(){
    console.log("server started at 3003");
})
module.exports=app;

//--------------------------- dbconnect.js -------------------------------<br>
const mysql = require('mysql2')
const mysqlConnection = mysql.createConnection(
{
   host:'localhost',
   user:'root',
   password:'root',
   database:'wpt',
   port:3306

})
mysqlConnection.connect((error)=>{
    if(error)
    {
        console.log(error);
    }
    else{
        console.log("connected")
    }
})
module.exports=mysqlConnection;

//------------------------ addcourse.ejs ------------------------------<br>
 <form action="/insertcourse" method="post">
        Course Id:<input type="text" name="cid" id="cid"><br>
        Course Name:<input type="text" name="cname" id="cname"><br>
        Course Fees:<input type="text" name="fees" id="fees"><br>
        Course Duration:<input type="text" name="duration" id="duration"><br>
        <button type="submit" name="btn" id="btn">Add Course</button>
    </form>

//---------------------- updatecourse.ejs------------------------------<br>
 <form action="/updatecourse" method="post">
        Course Id:<input type="text" name="cid" id="cid" value="<%=editdata.cid%>" readonly><br>
        Course Name:<input type="text" name="cname" id="cname" value="<%=editdata.cname%>"><br>
        Course Fees:<input type="text" name="fees" id="fees" value="<%=editdata.fees%>"><br>
        Course Duration:<input type="text" name="duration" id="duration" value="<%=editdata.duration%>"><br>
        <button type="submit" name="btn" id="btn">Update Course</button>
    </form>

//----------------------- displaycourse.ejs--------------------------<br>
    <form action="/addcourseform">
        <button type="submit" name="btnadd" id="add" value="add">Add New Course</button>
    </form>
    <table border="1">
        <tr>
            <th>Course Id</th>
            <th>Course Name</th>
            <th>Course fees</th>
            <th>Course Duration</th>
        </tr>
        <% for(var i=0;i<coursdata.length;i++){%>
            <tr>
                <td><%=coursdata[i].cid%></td>
                <td><%=coursdata[i].cname%></td>
                <td><%=coursdata[i].fees%></td>
                <td><%=coursdata[i].duration%></td>
                <td>
                    <a href="deletecourse/<%=coursdata[i].cid%>">Delete</a>
                    <a href="editcourse/<%=coursdata[i].cid%>">Edit</a>
                </td>
            </tr>
        <%}%>
    </table>
    <form action="/logout">
        <button type="submit">logout</button>
    </form>

   //-------------------------------- login.ejs -----------------------------<br>
 <form action="/login" method="post">
        <label>Email : 
            <input type="email" name="email" id="email">
        </label><br>
        <label>Password:
            <input type="password" name="password" id="password">
        </label><br>
        <button type="submit">Login</button>
    </form>
