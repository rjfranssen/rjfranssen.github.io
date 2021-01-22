# in .Rprofile of the website project
# ref: https://alison.rbind.io/post/2019-02-21-hugo-page-bundles/

if (file.exists("~/.Rprofile")) {
  base::sys.source("~/.Rprofile", envir = environment())
}

options(blogdown.new_bundle = TRUE)

options(
  blogdown.author = "rjfranssen",
  blogdown.ext = ".Rmarkdown",
  blogdown.subdir = "post",
  blogdown.yaml.empty = TRUE,
  blogdown.new_bundle = TRUE,
  blogdown.title_case = TRUE,
  blogdown.hugo.version = "0.79.0",
  blogdown.knit.on_save = TRUE,
  #blogdown.server.verbose = TRUE, # new in 1.1
  blogdown.files_filter = blogdown:::filter_md5sum
)
