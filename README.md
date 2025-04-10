          a!localVariables(
            /* Find children of the current parentFolderId */
            local!children: a!forEach(
              items: folders,
              expression: if(
                fv!item.parentFolderId = parentFolderId,
                {
                  id: fv!item.id,
                  folderName: fv!item.folderName,
                  parentFolderId: fv!item.parentFolderId,
                  children: rule!buildFolderTree(folders, fv!item.id)
                },
                null
              )
            ),
            
            /* Remove any null values from the result */
            local!children: filter(
              x -> x != null,
              local!children
            ),
            
            /* Return the children */
            local!children
          )
