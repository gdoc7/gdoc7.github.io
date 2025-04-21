---
title: Obsidian UI en WSL
tags: [WSL, Obsidian, Neovim]
style: 
color: primary
description: Usando la UI de Obsidian en Windows WSL.
---
## Instalación

Para usar Obsidian UI en WSL existen varios métodos, en este post estaremos usando la versión portable de Obsidian que sería el formato [AppImage](https://appimage.org/) en la ultima versión estable de Obsidian, para la fecha de este post sería la versión 1.8.9.

1. Descargar .AppImage de Obsidian:
```bash
wget https://github.com/obsidianmd/obsidian-releases/releases/download/v1.8.9/Obsidian-1.8.9.AppImage
```

2. Extraer el contenido del .AppImage:
```bash
./Obsidian-1.8.9.AppImage --appimage-extract
```

3. Ingresamos a la carpeta y finalmente ejecutamos el AppRun:
```bash
cd squashfs-root
./AppRun #Ejecutamos Obsidian
```

> Revisar versiones recientes de [obsidian](https://github.com/obsidianmd/obsidian-releases) 


Finalmente podemos agregar a nuestro fichero de `.zshrc` o `.bashrc` el siguiente alias: 
```bash
alias obsidian="nohup /Ruta/a/tu/carpeta/squashfs-root/AppRun > /dev/null 2>&1 &"
```

Ahora al usar el comando `obsidian` en la terminal, se estaría ejecutando en segundo plano la aplicación y podemos seguir trabajando en la terminal mientras usamos Obsidian.

## Usos
Desde la UI podemos observar nuestras notas creadas desde otras fuentes. En mi caso en la siguiente imagen podemos observar como se ven las notas creadas desde mi plugin de obsidian en Neovim.

{% include elements/figure.html image="../assets/obsidian-ss.jpg" caption="Vision grafica de las notas creadas con Neovim" %}

También podemos crear nuevas notas y linkearlas entre ellas desde la UI. Todas las funciones desde la UI estan disponibles y como estamos trabajando en el mismo vault podemos editarlas directamente en la UI o desde Neovim y podemos observar los cambios en tiempo real.


{% include elements/figure.html image="../assets/obsidian-note.jpg" caption="Nota creada con la UI" %}

{% include elements/figure.html image="../assets/obsidian-graph.jpg" caption="Visión grafica de los links desde la UI" %}

## Extras
#### Cómo usar Obsidian desde Neovim
Para usar Obsidian desde Neovim con LazyVim primero debemos crear la siguiente estructura de carpetas en el Home Path ($HOME):
```Bash
📁vaults
└── 📁 work
    ├── 📁files
    ├── 📁limbo
    └── 📁templates
```

Una vez creada la estructura de carpetas para el vault y siguiento la [Estructura de carpetas](https://www.lazyvim.org/configuration) de LazyVim crearemos un fichero dentro de la carpeta plugin llamado `obsidian.lua` con la configuración para activar el plugin obsidian.nvim, siguiendo el siguiente fichero
```lua

-- This file contains the configuration for the obsidian.nvim plugin in Neovim.

return {
  {
    -- Plugin: obsidian.nvim
    -- URL: https://github.com/epwalsh/obsidian.nvim
    -- Description: A Neovim plugin for integrating with Obsidian, a powerful knowledge base that works on top of a local folder of plain text Markdown files.
    "epwalsh/obsidian.nvim",
    version = "*", -- Use the latest release instead of the latest commit (recommended)

    dependencies = {
      -- Dependency: plenary.nvim
      -- URL: https://github.com/nvim-lua/plenary.nvim
      -- Description: A Lua utility library for Neovim.
      "nvim-lua/plenary.nvim",
    },

    opts = {
      -- Define workspaces for Obsidian
      workspaces = {
        {
          name = "work", -- Name of the workspace
          path = "~/vaults/work", --Path to the notes directory
        },
      },
      -- Completion settings
      completion = {
        min_chars = 2,
      },

      notes_subdir = "limbo", -- Subdirectory for notes
      new_notes_location = "limbo", -- Location for new notes

      -- Settings for attachments
      attachments = {
        img_folder = "files", -- Folder for image attachments
      },

      -- Settings for daily notes
      daily_notes = {
        template = "note", -- Template for daily notes
      },

      -- Key mappings for Obsidian commands
      mappings = {
        -- "Obsidian follow"
        ["<leader>of"] = {
          action = function()
            return require("obsidian").util.gf_passthrough()
          end,
          opts = { noremap = false, expr = true, buffer = true },
        },
        -- Toggle check-boxes "obsidian done"
        ["<leader>od"] = {
          action = function()
            return require("obsidian").util.toggle_checkbox()
          end,
          opts = { buffer = true },
        },
        -- Create a new newsletter issue
        ["<leader>onn"] = {
          action = function()
            return require("obsidian").commands.new_note("Newsletter-Issue")
          end,
          opts = { buffer = true },
        },
        ["<leader>ont"] = {
          action = function()
            return require("obsidian").util.insert_template("Newsletter-Issue")
          end,
          opts = { buffer = true },
        },
      },

      -- Function to generate frontmatter for notes
      note_frontmatter_func = function(note)
        -- This is equivalent to the default frontmatter function.
        local out = { id = note.id, aliases = note.aliases, tags = note.tags }

        -- `note.metadata` contains any manually added fields in the frontmatter.
        -- So here we just make sure those fields are kept in the frontmatter.
        if note.metadata ~= nil and not vim.tbl_isempty(note.metadata) then
          for k, v in pairs(note.metadata) do
            out[k] = v
          end
        end
        return out
      end,

      -- Function to generate note IDs
      note_id_func = function(title)
        -- Create note IDs in a Zettelkasten format with a timestamp and a suffix.
        -- In this case a note with the title 'My new note' will be given an ID that looks
        -- like '1657296016-my-new-note', and therefore the file name '1657296016-my-new-note.md'
        local suffix = ""
        if title ~= nil then
          -- If title is given, transform it into valid file name.
          suffix = title:gsub(" ", "-"):gsub("[^A-Za-z0-9-]", ""):lower()
        else
          -- If title is nil, just add 4 random uppercase letters to the suffix.
          for _ = 1, 4 do
            suffix = suffix .. string.char(math.random(65, 90))
          end
        end
        return tostring(os.time()) .. "-" .. suffix
      end,

      -- Settings for templates
      templates = {
        subdir = "templates", -- Subdirectory for templates
        date_format = "%Y-%m-%d-%a", -- Date format for templates
        gtime_format = "%H:%M", -- Time format for templates
        tags = "", -- Default tags for templates
      },
    },
  },
}

```

Con esta configuración ya podrías usar obsidian tanto desde la UI o desde Neovim con Windows WSL, recomiendo que visites el repositorio de [obsidian.nvim](https://github.com/epwalsh/obsidian.nvim) para mayor personalización. 
