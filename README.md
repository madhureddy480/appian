                              a!forEach(
                              items: folderList,
                              expression: a!boxLayout(
                              contents: {
                              a!richTextDisplayField(
                              value: {
                              a!richTextItem(
                                text: repeat(count(index(fv!item, "parentFolderId", {})), "⎯ ") & fv!item.folderName,
                                link: a!dynamicLink(
                                  label: fv!item.folderName,
                                  value: fv!item.id,
                                  saveInto: local!selectedFolder
                                ),
                                linkStyle: if(local!selectedFolder = fv!item.id, "STRONG", "STANDALONE")
                              ),
                              if(
                                length(fv!item.children) > 0,
                                a!richTextItem(
                                  text: if(contains(expandedIds, fv!item.id), "[-]", "[+]"),
                                  link: a!dynamicLink(
                                    value: fv!item.id,
                                    saveInto: {
                                      if(contains(expandedIds, fv!item.id),
                                        a!save(expandedIds, remove(expandedIds, fv!item.id)),
                                        a!save(expandedIds, append(expandedIds, fv!item.id))
                                      )
                                    }
                                  )
                                ),
                                {}
                              )
                              }
                              ),
                              if(
                              contains(expandedIds, fv!item.id),
                              rule!renderFolderTree(fv!item.children, expandedIds, selectedId),
                              {}
                              )
                              }
                              )
                              )
