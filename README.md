# appian

a!localVariables(
  /* Query Folders from Record Type */
  local!folders: a!queryRecordType(
    recordType: recordType!Folder,
    filters: {},
    pagingInfo: a!pagingInfo(startIndex: 1, batchSize: 1000)
  ).data,
  
  /* Build the folder tree recursively */
  local!folderTree: rule!buildFolderTree(local!folders, null),
  
  /* State variables */
  local!expandedIds: {},  /* Track expanded folders */
  local!selectedFolder: null,  /* Track selected folder */
  local!newFolderName: "",  /* Track new folder name */
  local!renameFolderName: "",  /* Track folder name to rename */
  
  /* Interface layout */
  a!formLayout(
    label: "Folder Manager",
    contents: {
      a!sectionLayout(
        label: "Folder Tree",
        contents: {
          /* Render the folder tree recursively */
          rule!renderFolderTree(local!folderTree, local!expandedIds, local!selectedFolder)
        }
      ),
      a!sectionLayout(
        label: "Add New Folder",
        contents: {
          a!textField(
            label: "New Folder Name",
            value: local!newFolderName,
            saveInto: local!newFolderName
          ),
          a!buttonLayout(
            primaryButtons: {
              a!buttonWidget(
                label: "Add Folder",
                saveInto: {
                  /* Add new folder logic */
                  a!save(local!folders, append(
                    local!folders,
                    a!map(
                      id: rand(),  /* For demo purposes, use rand() for ID */
                      folderName: local!newFolderName,
                      parentFolderId: local!selectedFolder
                    )
                  )),
                  a!save(local!newFolderName, "")  /* Clear the new folder name */
                },
                style: "PRIMARY"
              )
            }
          )
        }
      ),
      a!sectionLayout(
        label: "Rename Folder",
        contents: {
          a!textField(
            label: "New Name",
            value: local!renameFolderName,
            saveInto: local!renameFolderName
          ),
          a!buttonLayout(
            primaryButtons: {
              a!buttonWidget(
                label: "Rename Selected",
                disabled: isnull(local!selectedFolder),
                saveInto: {
                  /* Rename folder logic */
                  a!save(
                    local!folders,
                    a!forEach(
                      items: local!folders,
                      expression: if(
                        fv!item.id = local!selectedFolder,
                        a!map(
                          id: fv!item.id,
                          folderName: local!renameFolderName,
                          parentFolderId: fv!item.parentFolderId
                        ),
                        fv!item
                      )
                    )
                  ),
                  a!save(local!renameFolderName, "")  /* Clear the rename folder name */
                },
                style: "PRIMARY"
              )
            }
          )
        }
      )
    },
    buttons: a!buttonLayout(
      primaryButtons: {
        a!buttonWidget(
          label: "Submit",
          style: "PRIMARY",
          /* Add DB update logic here for persistence */
          saveInto: {}
        )
      }
    )
  )
)
