# in .Rprofile of the website project
# ref: https://alison.rbind.io/post/2019-02-21-hugo-page-bundles/

if (file.exists("~/.Rprofile")) {
  base::sys.source("~/.Rprofile", envir = environment())
}

options(blogdown.new_bundle = TRUE)

options(
  blogdown.author = "rjfranssen",
  blogdown.ext = ".Rmd",
  blogdown.subdir = "post",
  blogdown.yaml.empty = TRUE,
  blogdown.new_bundle = TRUE,
  blogdown.title_case = TRUE,
  blogdown.hugo.version = "0.79.1",
  blogdown.knit.on_save = FALSE,
  blogdown.files_filter = blogdown:::md5sum_filter
)
