library(reticulate)
library(httr)

# Install Python dependencies (only required if not installed)
py_install("atproto", pip = TRUE)

# Import Bluesky API
bs <- import("atproto")

# Retrieve Bluesky login credentials securely from GitHub Secrets
username <- Sys.getenv("BLUESKY_USERNAME")  
app_password <- Sys.getenv("BLUESKY_APP_PASSWORD")

# Authenticate Bluesky session
session <- bs$Client()
session$login(username, app_password)
print("✅ Successfully logged into Bluesky!")

# List of reporters' Bluesky handles
reporters <- c("adamschefter-mirror.bluesky.bot", "rapsheet1.bsky.social",
               "tompelissero.bsky.social", "nfldraftscout.bsky.social", "fieldyates.bsky.social", 
               "mike-yam.bsky.social", "wyche89.bsky.social", "cameronwolfe.bsky.social",
               "nflscorigami.bsky.social")

# Create an empty list to store posts
all_posts <- list()

# Loop through each reporter and fetch their latest post
for (reporter in reporters) {
  print(paste("📡 Fetching posts from:", reporter))

  # Fetch the latest post
  params <- dict(actor = reporter)
  posts <- session$app$bsky$feed$get_author_feed(params)
  
  # Extract posts safely
  if ("feed" %in% names(posts) && length(posts$feed) > 0) {
    all_posts[[reporter]] <- posts$feed[[1]]$post$record$text
    print(paste("📰 Latest post from", reporter, ":", all_posts[[reporter]]))
  } else {
    all_posts[[reporter]] <- "(No new updates)"
    print(paste("⚠️ No feed data found for", reporter))
  }
}

print("✅ Fetched all reporters' posts!")

# Retrieve Discord Webhook URL securely from GitHub Secrets
discord_webhook <- Sys.getenv("DISCORD_WEBHOOK_URL")

# Loop through stored posts and send to Discord
for (reporter in names(all_posts)) {
  latest_post <- all_posts[[reporter]]

  # Format message
  msg <- list(content = paste0("📰 **Latest from ", reporter, ":**\n", latest_post))
  
  # Send to Discord
  response <- POST(discord_webhook, body = msg, encode = "json", 
                   add_headers(`Content-Type` = "application/json"))

  print(paste("✅ Sent", reporter, "post to Discord! Status:", response$status_code))
}
