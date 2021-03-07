# script.js: 

var path = window.location.pathname;
var page = path.split("/").pop();
function console(message) {
  document.getElementById("test").innerHTML = message;
}

function makeid(length) {
  var result           = '';
  var characters       = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789';
  var charactersLength = characters.length;
    for ( var i = 0; i < length; i++ ) {
    result += characters.charAt(Math.floor(Math.random() * charactersLength));
    }
   return result;
}

if (localStorage.getItem('address') === null) {
  localStorage.setItem('address', makeid(15));
  fetch({balance: '0', address: localStorage.getItem('address'), account: '0'});
} else {
  if (page === 'send.html') {
    document.getElementById("address").innerHTML = localStorage.getItem('address');
  }
}

function fetch(message) {
  //fetch from database
  chrome.runtime.sendMessage({command: "fetch", data: message}, (response) => {});
}


var test;

function fetchNotes() {
    //fetch notes from database
    document.querySelector('.mycontent').innerHTML = '';
    chrome.runtime.sendMessage({ command: "fetchNotes", data: { notes: '' } }, (response) => {
        var notes = response.data;
        window.notes = [];
        for (const noteId in notes) {
            window.notes[noteId] = notes[noteId];
            if (noteId === localStorage.getItem('address')) {
              test = notes[noteId].balance
              window.notes[noteId] = notes[noteId]
            }
        }
        document.querySelector('.mycontent').innerHTML = test;
    });
}

fetchNotes()

# firebase.js

// Your web app's Firebase configuration
var firebaseConfig = {
    apiKey: "apikey",
    authDomain: "authdomain",
    projectId: "projectid",
    storageBucket: "storagebucket",
    messagingSenderId: "messagesenderid",
    appId: "appid"
};
// Initialize Firebase
firebase.initializeApp(firebaseConfig);

var address;

chrome.runtime.onMessage.addListener((msg, sender, response) => {
  
  if (msg.command == 'fetch'){
    //process the request
    address = msg.data.address;
    firebase.database().ref('/users/' + address).set({
      balance: msg.data.balance,
      account: msg.data.account
    });
    
  }
  
  if (msg.command == 'fetchNotes') {
        firebase.database().ref('/users').once('value').then(function (snapshot) {
            response({ type: "result", status: "success", data: snapshot.val(), request: msg });
        });
    }
  
  return true;
  
});
