# Custom Semantic Search 

You can use it to power advanced search across any YouTube channel or playlist. 

How to get started

- Clone the repository to your local machine.
Navigate to the root directory of the repository in your terminal.
- use node 18+
- Run the command npm install to install all the necessary dependencies.
- a file would be created in the src/bin/out folder, copy that file in the root /out/ folder
- Run the command npx tsx src/bin/resolve-yt-playlist.ts to download the English transcripts for each episode of the target playlist (in this case, the All-In Podcast Episodes Playlist).
- Run the command npx tsx src/bin/process-yt-playlist.ts to pre-process the transcripts and fetch embeddings from OpenAI, then insert them into a Pinecone search index.
- You can now run the command npx tsx src/bin/query.ts to query the Pinecone search index.
- (Optional) Run the command npx tsx src/bin/generate-thumbnails.ts to generate timestamped thumbnails of each video in the playlist. This step takes ~2 hours and requires a stable internet connection.


The frontend of the project is a Next.js webapp deployed to Vercel that uses the Pinecone index as a primary data store. You can run the command npm run dev to start the development server and view the webapp locally.
Note that a few episodes may not have automated English transcriptions available, and that the project uses a hacky HTML scraping solution for this, so a better solution would be to use Whisper to transcribe the episode's audio. Also, the project support sorting by recency vs relevancy.

## How It Works
Under the hood, it uses:

- OpenAI - We're using the brand new text-embedding-ada-002 embedding model, which captures deeper information about text in a latent space with 1536 dimensions
This allows us to go beyond keyword search and search by higher-level topics.
- Pinecone - Hosted vector search which enables us to efficiently perform k-NN searches across these embeddings
- Next.js - React web framework
- We use Node.js and the YouTube API v3 to fetch the videos of our target playlist. In this case, we're focused on the All-In Podcast Episodes Playlist, which contains 108 videos at the time of writing.

 Note that a few episodes don't have automated English transcriptions available, so we're just skipping them at the moment. A better solution would be to use Whisper to transcribe each episode's audio.

Once we have all of the transcripts and metadata downloaded locally, we pre-process each video's transcripts, breaking them up into reasonably sized chunks of ~100 tokens and fetch it's text-embedding-ada-002 embedding from OpenAI. This results in ~200 embeddings per episode.

All of these embeddings are then upserted into a Pinecone search index with a dimensionality of 1536. There are ~17,575 embeddings in total across ~108 episodes of the All-In Podcast.

npx tsx src/bin/process-yt-playlist.ts
Once our Pinecone search index is set up, we can start querying it either via the webapp or via the example CLI:

npx tsx src/bin/query.ts
We also support generating timestamp-based thumbnails of every YouTube video in the playlist. Thumbnails are generated using headless Puppeteer and are uploaded to Google Cloud Storage. We also post-process each thumbnail with lqip-modern to generate nice preview placeholder images.

