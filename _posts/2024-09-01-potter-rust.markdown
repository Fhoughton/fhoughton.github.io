---
layout: post
title:  "Rewriting My PS1 Harry Potter Game Files Extractor In Rust"
date:   2024-09-01 00:19:13 +0100
categories: Rust psx ps1
---
# What Is Hagrid?
This week I rewrote my tool '[hagrid](https://github.com/Fhoughton/hagrid/tree/master)' in Rust. Hagrid is a tool for extracting the data files from the Playstation 1 Harry Potter video games, famous for their good gameplay and funny looking characters (see the much memed PS1 Hagrid below).

![](/images/potter-hagriddemorust.gif)

<small> <i> A short demo of hagrid extracting and repacking the game files </i> </small>

<img src="/images/potter-ps1hagrid.jpg" width="400"/>

<small> <i> The notorius PS1 Hagrid model </i> </small>

# Why Rewrite It In Rust?
Quite simply I rewrote the tool because the C version was an unmaintainable mess, and I knew it needed a rewrite when it became a struggle to implement new file types into the existing argument parser/program. It had terrible bugs in its argument parser, some small bugs with the file repacking, and was generally lacking in code clarity. The goal of rewriting the code in Rust was to create a much more maintainable and simplistic program, that levarged Rust's more modern outlook for issues like file handling and argument parsing.

# Great Improvements

**Argument Parsing**

The rewrite created some great improvements to the program flow. Using the Rust [clap](https://docs.rs/clap/latest/clap/) library as a replacement for [getopt](https://www.gnu.org/software/libc/manual/html_node/Example-of-Getopt.html) was a lovely experience. Whereas in getopt I still had to handle parsing the flags the user entered into meaningful states from toggles I was able to use structs with clap and deserialise the arguments into the structs for access at runtime.

Here is the C argument parsing code:
```c
int main(int argc, char *argv[]) {
    FileMode file_mode = EXTRACT;
    int opt;

    while ((opt = getopt(argc, argv, "ep")) != -1) {
        switch (opt) {
        case 'e': file_mode = EXTRACT; break;
        case 'p': file_mode = PACK; break;
        default:
            print_usage(argv);
            exit(EXIT_FAILURE);
        }
    }

    // Check enough arguments were given
    switch (file_mode) {
        case EXTRACT:
            if ((argc - optind) < 3) {
                print_usage(argv);
                exit(EXIT_FAILURE);
            }
            datdir_extract(argv[optind], argv[optind+1], argv[optind+2]);
            break;
        case PACK:
            // hagrid -p <path> <filename_out>
            printf("Pack!\n");
            datdir_repack(argv[optind], argv[optind+1], argv[optind+2]);
            break;
        default:
            print_usage(argv);
            exit(EXIT_FAILURE);
    }
}
```

And here is the much prettier Rust version:
```rust
fn main() {
    let args = Args::parse();

    match args.cmd {
        Commands::DatDir {
            mode,
            dat_file,
            dir_file,
            path,
        } => {
            match mode {
                // e.g. hagrid dat-dir --mode extract --dat-file potter01_extracted/POTTER.DAT --dir-file potter01_extracted/POTTER.DIR --path out
                FileMode::Extract => {
                    datdir_extract(dat_file, dir_file, path);
                }

                // e.g. hagrid dat-dir --mode pack --dat-file MYPOTTER.dat --dir-file MYPOTTER.dir --path out
                FileMode::Pack => {
                    datdir_pack(dat_file, dir_file, path);
                }
            }
        }
    }
}
```

As you can see the Rust code was more clear, abstract and shorter (24 vs 34 lines), a theme that was present throughout the entire rewriting experience.

**Byte Ordering**

Whilst somewhat anecdotal I appreciated that all of the Rust apis for converting bytes to and from integer types were explicit, and that all Rust integer types were explicit about their signedness and size. I had previously had to comb the C codebase to make sure all integers were using the stdint.h sizes (uint32_t etc.) and was never quite confident I got them all, meanwhile Rust's explicit nature for type sizes and conversions made me feel much more comfortable that I had gotten it right and that my code was truly portable and correct.

```rust
/* Dir file info reading */
// 4 byte filesize (in bytes)
let filesize_bytes = &dir_contents[dir_offset + 12..dir_offset + 16];
let filesize = u32::from_le_bytes(filesize_bytes.try_into().unwrap()) as usize;

// 4 byte file offset
let offset_bytes = &dir_contents[dir_offset + 16..dir_offset + 20];
let dat_offset = u32::from_le_bytes(offset_bytes.try_into().unwrap()) as usize;
```

Whilst in C I copied bytes direcltly and cast them at will, which could've gone wrong on a different plaform due to endianness:

```c
// Copy bytes from file directly to the FileEntry struct for reading
#pragma pack(push, 1) // Ensure there is no padding in the struct
typedef struct {
    char filename[12]; // Maximum 11 characters and space for a null-terminator
    uint32_t filesize; // How many bytes to read
    uint32_t offset; // Where to read from in dat file
} FileEntry;
#pragma pack(pop)

fread(&entry, sizeof(FileEntry), 1, dir_fp); // This could silently go wrong if the uint32_t on another platform was big endian!
```

Just looking at the casting code there are so many sources of bugs that could creep in and cause issues or damage portability, something I feel comfortable about not facing with Rust.

**Speed**

The Rust version of [hagrid](https://github.com/Fhoughton/hagrid/blob/master/src/main.rs) is much faster than its C counterpart. This is likely due to the way files are read, with the Rust code reading the entire files all at once, whilst the C code reads them in byte chunks (as it was easier to do each technique in their respective language). Both have drawbacks but in this case as we are handling fairly small files (no more than 200MB) the Rust technique greatly won out.

# My Overall Feelings After Rewriting It In Rust
My experience was extremely positive, and I think the final program is far better compared to the C version that came before it. I chose to use C originally because it is fun to use but the stress-free nature of Rust and the value it brings was a charm. I'm unsure about wider movements to rewrite all code in Rust, as some code simply seems too specific or far gone to be changed (such as embedded devices or code tied strongly to C style apis such as SDL), but that doesn't seem to concern bigger people than me, such as DARPA who have recently launched [TRACTOR](https://www.darpa.mil/program/translating-all-c-to-rust) (Translate All C To Rust), a government program to fund the conversion of as much government C code as possible to safe rust. However, I do feel I achieved my aims either way, and I can hopefully add support for other formats used by the game in the near future.

# An Aside: Doom Emacs
I'm just going to end this post with an update on my Doom Emacs experience.

I rewrote the codebase and this blog post in Doom Emacs, which I mentioned giving a try in [Week 6](https://fhoughton.github.io/lua/refactoring/emacs/2024/08/17/week6-lua-refactor.html) as an alternative to Visual Studio Code.

I found the experience very pleasant and I continue to be a big fan of Doom Emacs. I was able to get the Rust LSP set up by simply changing my config files (~/.doom.d/init.el) to enable Rust and then refreshing the config (SPC-h-r-r), at which point I was asked if I wanted to install the LSP, and after choosing yes I was up and running to write Rust code with full support from the IDE. This was a smooth process, and a breath of fresh air from some VSCode extensions which make you install the language server yourself, and then fight you over recognizing it exists or what version it's on compared to the extensions expectations.

There is still a lot for me to adjust to, I'm still not happy with the workflow with the terminal (it doesn't position or feel right), with compiling commands (the compilation buffer is annoying and doesn't interact with the terminal well) and the file browsing (dired feels unintuitive) but these feel like issues that can be ironed out with config tweaks and learning time. I also miss some features, such as VSCode helping me find the paths for images when linking them within Markdown documents.

I feel much faster already, especially with good Vim bindings and fast file management keybinds (SPC-f-r for recent files and SPC-. for files in the directory I'm editing in). I hope that with time as I adjust I can create a more seamless experience and gain a stronger advantage from my new choice of tooling.

I am also excited by the prospect of org mode, and have been trying out org-agenda a little bit (I haven't switched to it yet, just given it a go to see how it works). It feels like I have a lot to gain from getting deeper into the Doom Emacs systems, and I'm excited to see more. It's a shame that some powerful features like org-agenda feel poorly integrated with the outside world however (it's a huge struggle to add my Google Calendar from work to my agenda calendar!).
