from flask import Flask, render_template, redirect, session, request, url_for
from flask_socketio import join_room, leave_room, SocketIO, send
import random as r
from string import ascii_uppercase

app = Flask(__name__)
app.config['SECRET_KEY'] = 'super_secret_key'
socketio = SocketIO(app)

rooms = {}

def generate_unique_code(length):
    while True:
        code = "".join(r.choices(ascii_uppercase, k=length))
        if code not in rooms:
            break
    return code

@app.route('/', methods=['GET', 'POST'])
def home():
    session.clear()
    if request.method == "POST":
        name = request.form.get('name')
        code = request.form.get('code')
        join = request.form.get('join', False)
        create = request.form.get('create', False)

        if not name:
            return render_template('home.html', error="Please enter your name!", name=name, code=code)

        if join != False and not code:
            return render_template('home.html', error="Please enter a room code!", name=name, code=code)

        room = code
        if create != False:
            room = generate_unique_code(4)
            rooms[room] = {"members": 0, "messages": []}
        elif code not in rooms:
            return render_template('home.html', error="Room does not exist.", name=name, code=code)

        session["name"] = name
        session["room"] = room
        return redirect(url_for("room"))

    return render_template('home.html')

@app.route('/room')
def room():
    room = session.get("room")
    if room is None or session.get("name") is None or room not in rooms:
        return redirect(url_for("home"))
    return render_template('room.html', code=room, messages=rooms[room]["messages"])

@socketio.on("message")
def message(data):
    room = session.get("room")
    if not room or room not in rooms:
        return

    content = {
        "name": session.get("name"),
        "message": data["data"]
    }
    send(content, to=room)
    rooms[room]["messages"].append(content)
    print(f"{session.get('name')} said: {data['data']}")

@socketio.on("connect")
def connect(auth):
    room = session.get("room")
    name = session.get("name")
    if not room or not name:
        return
    if room not in rooms:
        leave_room(room)
        return

    join_room(room)
    send({"name": name, "message": "has entered the room"}, to=room)
    rooms[room]["members"] += 1
    print(f'{name} joined room {room}')

@socketio.on("disconnect")
def disconnect():
    room = session.get("room")
    name = session.get("name")

    if room in rooms:
        rooms[room]["members"] -= 1
        if rooms[room]["members"] <= 0:
            del rooms[room]

    send({"name": name, "message": "has left the room"}, to=room)
    print(f"{name} has left the room {room}")

if __name__ == "__main__":
    socketio.run(app, debug=True, port=2004)

# js code for change icon



        let ans = document.getElementById("icon")
        ans.style.backgroundColor = "Blue"
        // let icon = document.getElementById('icon')
        // // char = ['A','B','C','D']
        // // if(name[0] == char[0]){
        // icon.style.backgroundColor = "red"
        //     console.log("done")
        // }