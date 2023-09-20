<script>
    import { page } from "$app/stores";
    import axios from "axios";
    import { onMount } from "svelte";
    let lastIndex = $page.url.toString().lastIndexOf("/");
    let id = $page.url.toString().substring(lastIndex + 1);
    let movieUrl = `https://api.themoviedb.org/3/movie/${id}/videos`;

    // Get the information of the movie
    let video;
    let ytbKey;
    let movieTitle;
    let movieGenres = [];
    let rating;
    let release_date;
    let image;
    let poster;
    let sypnosis;

    const options = {
        method: "GET",
        headers: {
            accept: "application/json",
            Authorization:
                "Bearer eyJhbGciOiJIUzI1NiJ9.eyJhdWQiOiI5MDFhMTUwYzViYWUwYWEwYjAwYzAwNTZjNmQwOTY2NSIsInN1YiI6IjY0YjJlNzZiNzg1NzBlMDExZGE4Zjc1YyIsInNjb3BlcyI6WyJhcGlfcmVhZCJdLCJ2ZXJzaW9uIjoxfQ.j823nGnvdRE6KT_1WzBJo5soKV0UrEvM1iRjTglEwnc",
        },
    };

    const movieInfos = async () => {
        const response = await axios.get(
            `https://api.themoviedb.org/3/movie/${id}`,
            options
        );
        poster = response.data.poster_path;
        image = `https://image.tmdb.org/t/p/w500${poster}`;
        movieTitle = response.data.title;
        movieGenres = response.data.genres;
        rating = response.data.vote_average;
        release_date = response.data.release_date;
        sypnosis = response.data.overview;
    };
    const getVideo = async () => {
        await movieInfos();
        (async () => {
            const response = await axios.get(movieUrl, options);
            const videoElement = response.data.results.find(
                (element) =>
                    element.type === "Trailer" || element.type === "Teaser"
            );
            console.log(response.data.results);

            if (videoElement) {
                ytbKey = videoElement.key;
                video = `https://www.youtube.com/watch?v=${ytbKey}`;
                console.log(ytbKey);
                console.log(videoElement.type);
            } else {
                console.log("pas trouver");
                console.log(response.data.results);
            }
        })();
    };

    onMount(() => getVideo());
</script>

<body>
    <div class="main-container">
        <div class="main-container">
            <div class="iframe-container">
                <iframe
                    src={`https://www.youtube.com/embed/${ytbKey}?autoplay=1&modestbranding=1`}
                />
            </div>
        </div>
    </div>
    <h3 class="title">{movieTitle}</h3>
    <div class="info-movie">
        <img class="movie-image" src={image} alt="" />
        <div class="left-info">
            <div class="container-genre">
                {#each movieGenres as genre}
                    <p class="genre-style">{genre.name}</p>
                {/each}
            </div>
            <p>Rating: {rating}/10</p>
            <p>Release date : {release_date}</p>
            <p>Sypnosis: {sypnosis}</p>
        </div>
    </div>
</body>

<style>
    @import url("https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Montserrat:ital,wght@0,400;0,700;1,400;1,600&display=swap");
    iframe {
        border-radius: 8px;
        border: none;
        width: 80vw;
        height: 50vh;
    }
    .iframe-container {
        display: flex;
        justify-content: center;
    }
    body {
        background-color: black;
        width: 100%;
        height: 100%;
        color: white;
        font-family: "Inter", sans-serif;
        font-size: 15px;
        line-height: 25px;
    }
    .main-container {
        display: flex;
        justify-content: center;
    }
    .title {
        color: #ffd700;
    }
    .info-movie {
        display: flex;
        height: fit-content;
        background-color: #282828;
        width: 100%;
    }
    .left-info {
        width: 100%;
        display: flex;
        flex-direction: column;
        margin-left: 12px;
        font-size: 1em;
    }
    .movie-image {
        width: 16rem;
        height: 21rem;
        border-radius: 5px;
    }
    .container-genre {
        display: flex;
        height: fit-content;
        justify-content: flex-start;
    }
    .genre-style {
        border: 1px white solid;
        border-radius: 3px;
        padding: 4px;
        margin-right: 10px;
    }
</style>
