<script>
import io from 'socket.io-client';

const socket = io('http://localhost:3000');

export default {
  name: "Films",
  data() {
    return {
      films:null,
      sallesCine:[],
      moviesInCine:[],
    }
  },
  mounted() {

      socket.on('sallecine', (rep) => {//receptionne les salles du cinema
        this.sallesCine=rep;
        this.getSeances();
      });
      socket.emit('sallecine',localStorage.getItem('idCinema'));//demande la liste des salles du cinema
  },
  methods:{
    //appel API GraphQL pour récupérer liste des films
    recupListeFilm(){
      fetch('http://localhost:4000/graphql', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          query: `
        query {
            movies {
                id_movie
                id_real
                id_genre1
                id_genre2
                duree
                titre
                image
              genre1 {
                id_genre
                nom_genre
              }
              genre2 {
                id_genre
                nom_genre
              }
              realisateur {
                id_realisateur
                nom_realisateur
              }
              }
        }
        `
        })
      }).then(response => response.json())
          .then(data => this.filtrerFilms(data.data.movies))
          .catch(error => console.error(error));
    },

    displayFilms(films){
      let nbfilms=0
      let nocol=1;
      let nolig=1
      let filmsdiv = document.getElementById("films");
      let currentLigne = document.createElement("div");
      currentLigne.classList.add("row","align-item-center");
      currentLigne.id="ligne"+nolig;
      films.forEach( (film) => {
        let newCol = document.createElement("div");
        newCol.classList.add("col-4");
        let imgfilm = document.createElement("img");
        imgfilm.src=film.image;
        imgfilm.style.width="100%";
        imgfilm.style.height="300px";
        let titre = document.createElement("h3");
        titre.textContent=film.titre;
        let real = document.createElement("p");
        real.textContent = "par : "+film.realisateur.nom_realisateur;
        let button = document.createElement("button");
        button.textContent="Voir séances";
        button.addEventListener('click', (e) => {//ajoute un event onClick au boutton
          localStorage.setItem('filmdata',JSON.stringify(film));
          localStorage.setItem('seances',JSON.stringify(this.getSeancesByFilm(film.id_movie)));
          location.assign("/film/"+film.id_movie);
        });
        //ajout du film dans la ligne
        newCol.appendChild(imgfilm);
        newCol.appendChild(titre);
        newCol.appendChild(real);
        newCol.appendChild(button);
        currentLigne.appendChild(newCol);
        if(nbfilms!=films.length-1){//si c'est pas le dernier film de la liste
          //creation d'une div colonne representant un film
          nocol++;
          if(nocol>3){//on met 3 films par ligne
            filmsdiv.appendChild(currentLigne);
            nolig++;
            let newLigne = document.createElement("div");
            newLigne.classList.add("row","align-item-center");
            newLigne.id="ligne"+nolig;
            currentLigne = newLigne;
            nocol=1;
          }
        }else{//si c'est le dernier film on insere la ligne

          filmsdiv.appendChild(currentLigne);
        }
        nbfilms++;

      })
    },

    //récupère toutes les séances appel à l'API REST
    async getSeances(){
      let response = await fetch('http://localhost:8081/sessions');
      let data = await response.json();
      //manque de synchro entre les db film et seance donc on fake les seances
      let fakeseances = this.getfakeSeances();
      let filtrerCine = {};
      for(let i=0;i<fakeseances.length;i++){
        if(this.sallesCine.includes(fakeseances[i].theater_id)){//verifie si les seances ont lieu dans les salles du cinéma
          this.moviesInCine.push(fakeseances[i].movie_id)
        }
      }
      this.recupListeFilm();
    },

    //filtre la liste des films pour afficher uniquement ceux dispo dans le cinema
    filtrerFilms(allFilms){
      let filtered = [];
      allFilms.forEach((film) => {
        if(this.moviesInCine.includes(film.id_movie)){
          filtered.push(film);
        }
      });
      this.displayFilms(filtered);
    },

    //retourne une liste de sessions avec des movie_id et theater_id valides (synnchro avec DB Film et reservation
    getfakeSeances(){
      const fake = [{"_id":"640600d92932bebe81b57401","movie_id":1,"theater_id":5,"date":"2023-01-25T00:00:00.000Z","start_time":"20:00"},{"_id":"640604d4369105e099381364","movie_id":2,"theater_id":6,"date":"2023-01-25T00:00:00.000Z","start_time":"20:00"},{"_id":"640604d8369105e099381365","movie_id":3,"theater_id":7,"date":"2023-01-25T00:00:00.000Z","start_time":"20:00"},{"_id":"642b125e996be353bc069b6c","movie_id":7,"theater_id":5,"date":"1970-01-01T00:00:00.000Z","start_time":"21:00"}];
      return fake;
    },

    //recupères les séances de diffusion du film
    getSeancesByFilm(idfilm){
      let allSeances = this.getfakeSeances();
      let newSeances = [];
      allSeances.forEach((seance) => {
        if(seance.movie_id==idfilm){
          newSeances.push(seance);
        }
      })
      return newSeances;
    }

  }
}
</script>

<template>

  <div class="container" id="films">
    <div class="row align-items-center">
      <h1>Les films du moment</h1></div>
  </div>
</template>

<style scoped>

</style>