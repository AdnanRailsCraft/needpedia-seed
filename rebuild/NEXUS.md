REBUILD INSTRUCTIONS — THE NEXUS DOCUMENT SERVER
===============================================
Part of rebuild/, the per-system rebuild instructions for the seed.
Start with rebuild/README.md for the overview.

The Nexus is Needpedia's document server: it serves reference articles,
the AI guides (thread posts), volunteer studio pages and the order/admin
pages. It runs entirely in Docker on any Linux machine.

WHAT YOU NEED
- A Linux computer with Docker and the Docker Compose plugin
  (`docker compose version` must work).
- About 1 GB of disk for the seed content plus images.

THE FOLDER
- code/nexus/ holds everything: the pages (web/), the small server
  program (order-server.py), the web server settings (nginx-default.conf)
  and the two settings files that start it all:
    docker-compose.yml       the main stack: web server, order server,
                             terminal, front door, Cloudflare Tunnel
    home-docker-compose.yml  an optional lighter stack: file browser +
                             front door only (no order server)

VOLUME PATHS (portable now)
- Both settings files use environment variables instead of one person's
  home folder:
    NEXUS_DIR  folder with the content to serve.
               Default: ./Needpedia_Nexus (a folder next to
               docker-compose.yml). Export it to point somewhere else:
                   export NEXUS_DIR=/path/to/your/content
    HOME       your home folder is mounted into the containers at
               /home/clearcrow (the original in-container path, kept so
               the existing scripts keep working).
- No editing of volume paths is required any more.

STEPS
1. Prepare the content folder. The pages live in web/, the content they
   serve (public thread posts, DEVLOG.md, images) lives elsewhere in the
   seed. Build a folder that holds all of it at the top level:

       cd code/nexus
       mkdir -p Needpedia_Nexus
       cp -r web/* Needpedia_Nexus/
       mkdir -p Needpedia_Nexus/public Needpedia_Nexus/data
       cp -r ../../media/images Needpedia_Nexus/images   # optional gallery
       # add thread-system posts, DEVLOG.md, articles as you want them served

   Anything in NEXUS_DIR is served at https://your-server/ — the web
   server refuses to serve code files, databases or dotfiles (see the
   deny rules in nginx-default.conf), and only handoff-style posts under
   public/ are blocked from anonymous download.

2. Set secrets:
       # in docker-compose.yml
       tunnel token  (line with "tunnel ... run --token REPLACE_ME"):
           get one from your Cloudflare Zero Trust dashboard, or delete
           the tunnel service and open port 8080 directly
       OPENROUTER_API_KEY (order-server environment):
           your own key from openrouter.ai (used by the AI studio pages)
       # in home-docker-compose.yml
       tunnel token likewise (REPLACE_ME== is a placeholder, not a suffix)

3. Start the main stack:
       cd code/nexus
       docker compose up -d
       docker compose ps          # all four services should be up

4. Check it:
       curl -I http://localhost:8080        # nginx answers
       curl http://localhost:8181/healthz   # order server answers (or /)
       open http://localhost:8080/gallery.html

5. Optional lighter stack (file browser instead of full stack):
       docker compose -f home-docker-compose.yml up -d
   Do not run both stacks at once — both want port 8080 and both create
   a container named cloudflare-tunnel.

6. Rebuild the folder listings and article index inside the content
   folder (scripts in code/nexus/): regen_index.py and
   write_folder_lists.py. They were written for the original author's
   machine layout (/home/n/...) — run them from your content folder and
   adjust the ROOT constants at the top of each script if needed.

KNOWN LIMITS (this edition of the seed)
- volunteers.json was left out — volunteer logins are yours to create.
- Videos, audio and mixed media were left out; images are in media/images/.
- order-server.py still writes activity numbers to
  /home/clearcrow/Nexus_private/... inside the container; on your
  machine that path maps to your own home folder (that is why HOME is
  mounted). Nothing else breaks if you ignore activity numbers.
- crontab.txt and the shell scripts reference the original author's
  paths (/home/n/...); adjust before putting them in cron.

TESTING THE SITE BEFORE ANYTHING ELSE
-------------------------------------
Not ready for a server? Run the site on your own computer first:
documents/developer-wiki/Needpedia-Test-Site-Package.md
(code/needpedia-test-sites, Docker, 4 GB memory).

ADDING MORE REBUILD FILES
-------------------------
One file per system, same shape as this one: what it is, what you need,
numbered steps, how to tell it worked, known limits. Tick its line in the
checklist in README.md when you add one.
