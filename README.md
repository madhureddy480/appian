    a!localVariables(
      local!children: a!forEach(
        items: wherecontains(parentFolderId, index(folders, "parentFolderId", {})),
        expression: {
          id: fv!item.id,
          folderName: fv!item.folderName,
          parentFolderId: fv!item.parentFolderId,
          children: rule!buildFolderTree(folders, fv!item.id)
        }
      ),
      local!children
    )
