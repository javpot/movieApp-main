<script>
  import { onMount } from "svelte";
  import { readable } from "svelte/store";
  import { writable } from "svelte/store";

  import axios from "axios";
  import { goto } from "$app/navigation";

  let apiKey = "901a150c5bae0aa0b00c0056c6d09665";
  let genresList = ["Movies", "TV shows", "Top 250"];
  let url = "https://api.themoviedb.org/3/movie/popular";
  let urlTV = "https://api.themoviedb.org/3/trending/tv/day";
  let isMenuOpen = false;
  let visibility = "hidden";

  let searchText;
  let margin = "-10vh";

  let h1 = "Movies";
  const options = {
    method: "GET",
    headers: {
      accept: "application/json",
      Authorization:
        "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI5MDFhMTUwYzViYWUwYWEwYjAwYzAwNTZjNmQwOTY2NSIsInN1YiI6IjY0YjJlNzZiNzg1NzBlMDExZGE4Zjc1YyIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.j823nGnvdRE6KT_1WzBJo5soKV0UrEvM1iRjTglEwnc",
    },
  };
  let movies = [];
  const fetchDataMovies = (typeGenre) => {
    if (typeGenre === "Movies") {
      url = "https://api.themoviedb.org/3/movie/popular";
    } else if (typeGenre === "Top 250") {
      url = "https://api.themoviedb.org/3/movie/top_rated";
    }
    (async () => {
      const response = await axios.get(url, options);
      movies = response.data.results.map((element) => {
        const fullYear = new Date(element.release_date);
        const year = fullYear.getFullYear();
        const image = `https://image.tmdb.org/t/p/w500${element.poster_path}`;
        return {
          id: element.id,
          title: element.title,
          poster_path: image,
          year: year,
          media_type: element.media_type,
        };
      });
      console.log(movies);
    })();
  };

  const fetchDataTV = async () => {
    const response = await axios.get(urlTV, options);
    movies = response.data.results.map((element) => {
      const fullYear = new Date(element.first_air_date);
      const year = fullYear.getFullYear();
      const image = `https://image.tmdb.org/t/p/w500${element.poster_path}`;
      return {
        id: element.id,
        title: element.name,
        poster_path: image,
        year: year,
        media_type: element.media_type,
      };
    });
    console.log("hello tv");
    console.log(movies);
  };

  const fetchMovieSearch = async () => {
    const response = await axios.get(
      `https://api.themoviedb.org/3/search/multi?query=${searchText}`,
      options
    );
    movies = response.data.results.map((element) => {
      if (element.media_type == "tv") {
        const fullYear = new Date(element.first_air_date);
        const year = fullYear.getFullYear();
        const image = `https://image.tmdb.org/t/p/w500${element.poster_path}`;

        return {
          id: element.id,
          title: element.name,
          poster_path: image,
          year: year,
          media_type: element.media_type,
        };
      } else {
        const fullYear = new Date(element.release_date);
        const year = fullYear.getFullYear();
        const image = `https://image.tmdb.org/t/p/w500${element.poster_path}`;

        return {
          id: element.id,
          title: element.title,
          poster_path: image,
          year: year,
          media_type: element.media_type,
        };
      }
    });
  };

  const menuOptionClicked = (/** @type {string} */ text) => {
    $: h1 = text;
    console.log(text);
    switch (text) {
      case "Movies":
        fetchDataMovies("Movies");

        break;
      case "TV shows":
        fetchDataTV();
        break;
      case "Genres":
        break;
      case "Top 250":
        fetchDataMovies("Top 250");
        break;
      default:
        break;
    }
  };

  function toggleMenu() {
    isMenuOpen = !isMenuOpen;
    if (isMenuOpen == true) {
      visibility = "visible";
      margin = "1em";
    } else {
      visibility = "hidden";
      margin = "-10vh";
    }
  }

  onMount(() => {
    fetchDataMovies("Movies");
  });

  // Je dois export ce id dans une autre page qui a pour but d'afficher les informations du film selectionner
  const movieData = (type, movieId) => {
    // Car quand la page mount le api call ne renvoi pas le type du film afficher. Il le renvoi seulement avec la requete de search
    if (type == undefined) {
      if (h1 == "Movies" || h1 == "Top 250") {
        type = "movie";
        window.location.href = `/description/movie/${movieId}`;
      } else {
        type = "tv";
        window.location.href = `/description/show/${movieId}`;
      }
    } else {
      if (type == "tv") {
        window.location.href = `/description/show/${movieId}`;
      } else {
        window.location.href = `/description/movie/${movieId}`;
      }
    }
  };

  const searchInputListener = (event) => {
    if (searchText == "") {
      fetchDataMovies();
    } else {
      console.log(searchText);
      fetchMovieSearch();
    }
  };
</script>

<body>
  <nav>
    <div class="left-navbar">
      <div class="title">
        <h2>Movizzz</h2>
      </div>

      <ul class="liste-genre">
        {#each genresList as genre}
          <a
            class="menu-option"
            href=""
            on:click={() => menuOptionClicked(genre)}><li>{genre}</li></a
          >
        {/each}
      </ul>
    </div>
    <div class="right-nav">
      <input
        class="inputSearch"
        type="text"
        placeholder="Search"
        bind:value={searchText}
        on:input={(event) => searchInputListener(event)}
      />
      <button class="menu-button" on:click={toggleMenu}
        ><img
          class="button-option"
          src="./src/img/icons8-utilisateur-40.png"
          alt=""
        /></button
      >
    </div>
  </nav>
  <div style="visibility: {visibility};" class="container-genre-mobile">
    <ul>
      {#each genresList as genre}
        <a
          class="menu-option-mobile"
          href=""
          on:click={() => menuOptionClicked(genre)}
          ><li style="visibility: {visibility};" class="li-mobile">
            {genre}
          </li></a
        >
      {/each}
    </ul>
  </div>
  <div class="main-container" style="margin-top: {margin};">
    <header>
      <hr />
      <h1>{h1}</h1>
      <hr />
    </header>
    <div class="movies-container">
      {#each movies as movie1, i (movie1.id)}
        {#if movie1.poster_path != "https://image.tmdb.org/t/p/w500undefined" && movie1.poster_path != "https://image.tmdb.org/t/p/w500null" && movie1.media_type != "person"}
          <div
            class="movie-card"
            on:click={() => movieData(movie1.media_type, movie1.id)}
          >
            <div class="movie-image">
              <a href="">
                <img class="movie-image" src={movie1.poster_path} alt="" /></a
              >
            </div>
            <div class="movie-title">{movie1.title}</div>
            <div class="movie-year">{movie1.year}</div>
          </div>
        {/if}
      {/each}
    </div>
  </div>
</body>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Montserrat:ital,wght@0,400;0,700;1,400;1,600&display=swap");
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
    font-family: "Inter", sans-serif;
  }
  body {
    background-color: #282828;

    width: 100vw;
    height: 100%;
  }
  .li-mobile {
    visibility: hidden;
  }
  nav {
    display: flex;
    background-color: rgba(15, 15, 15, 0.9);
    width: 100vw;
    height: 70px;
    align-items: center;
    justify-content: space-between;
  }
  .container-genre-mobile {
    display: none;

    background-color: rgba(15, 15, 15, 0.9);
    display: flex;
    align-items: center;
    flex-direction: column;
  }
  hr {
    color: gray;
    width: 65vh;
  }
  .left-navbar {
    display: flex;
    width: 70%;
    justify-content: space-around;
  }
  h2 {
    color: #ffd700;
  }
  .liste-genre {
    display: flex;
    justify-content: space-around;
    align-items: center;
    width: 50%;
    color: white;
  }

  .menu-option {
    color: inherit;
    text-decoration: none;
    list-style: none;
    text-decoration: underline 0.15em rgba(255, 255, 255, 0);
    transition: text-decoration-color 300ms;
  }
  .menu-option:hover {
    text-decoration-color: white;
  }
  .right-nav {
    display: flex;
    justify-content: space-between;
    margin-right: 40px;
  }
  header {
    margin-top: 5rem;
    display: flex;
    align-items: center;
    justify-content: space-evenly;
    color: #ffd700;
  }
  .movies-container {
    margin-top: 2em;
    border: 1px transparent solid;
    height: fit-content;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
    justify-content: space-evenly;
  }
  .movie-card {
    width: fit-content;
    height: fit-content;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin-bottom: 15px;
  }
  .movie-image {
    width: 18rem;
    height: 26rem;
  }
  .movie-title,
  .movie-year {
    text-align: center;
    color: white;
    width: 18rem;
    margin-top: 5px;
  }
  .inputSearch {
    background: rgb(81, 81, 81);
    border: none;
    height: 2rem;
    width: 13rem;
    color: white;
  }
  /* Responsive screen */
  .menu-button {
    display: none;
  }

  @media only screen and (max-width: 670px) {
    .menu-button {
      display: unset;
      border: none;
      appearance: none;
      background-color: transparent;
      width: 20px;
      margin-left: 8px;
    }

    .inputSearch {
      width: 12em;
      border-radius: 3px;
    }
    .left-navbar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      position: relative;
    }
    .right-nav {
      width: 40%;
    }

    h2 {
      margin-left: 25px;
    }
    .container-genre-mobile {
      height: 20vh;
      display: flex;
      background-color: rgba(15, 15, 15, 0.6);
    }
    ul {
      display: flex;
      flex-direction: column;
      display: flex;
      align-items: center;
      justify-content: space-around;
      height: 100%;
      width: 100%;

      color: white;
    }
    .liste-genre {
      visibility: hidden;
    }
    .menu-option-mobile {
      width: 100%;
      padding: 10px 5px;
      color: inherit;
      text-decoration: none;
      list-style: none;
      text-decoration: underline 0.15em rgba(255, 255, 255, 0);
      transition: 300ms;
      text-align: center;
      border: 1px transparent solid;
    }
    .menu-option-mobile:hover {
      background-color: rgba(15, 15, 15, 0.9);
    }
    .button-option {
      width: 25px;
    }
    h1 {
      font-size: 25px;
    }
    hr {
      width: 30%;
    }
  }
</style>
