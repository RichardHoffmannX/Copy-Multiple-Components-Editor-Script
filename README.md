# Copy-Multiple-Components-Editor-Script
Copy Multiple Components Editor Script in Unity

Save this script as CopyMultipleComponentsEditor.cs inside your Editor folder.

```
using UnityEngine;
using UnityEditor;
using System.Collections.Generic;

public class CopyMultipleComponentsEditor : EditorWindow
{
    private static List<Component> copiedComponents = new List<Component>();

    [MenuItem("Tools/Copy-Paste Multiple Components")]
    public static void ShowWindow()
    {
        GetWindow<CopyMultipleComponentsEditor>("Copy-Paste Components");
    }

    private void OnGUI()
    {
        GUILayout.Label("Copy & Paste Multiple Components", EditorStyles.boldLabel);

        if (GUILayout.Button("Copy Components from Selected GameObject"))
        {
            CopyComponents();
        }

        if (GUILayout.Button("Paste Components to Selected GameObject"))
        {
            PasteComponents();
        }
    }

    private static void CopyComponents()
    {
        if (Selection.activeGameObject == null)
        {
            Debug.LogWarning("No GameObject selected to copy from!");
            return;
        }

        copiedComponents.Clear();
        copiedComponents.AddRange(Selection.activeGameObject.GetComponents<Component>());

        // Remove the Transform component (since every GameObject must have one)
        copiedComponents.RemoveAll(c => c is Transform);

        if (copiedComponents.Count > 0)
        {
            Debug.Log($"Copied {copiedComponents.Count} components from {Selection.activeGameObject.name}");
        }
        else
        {
            Debug.LogWarning("No valid components to copy!");
        }
    }

    private static void PasteComponents()
    {
        if (copiedComponents.Count == 0)
        {
            Debug.LogWarning("No components copied!");
            return;
        }

        if (Selection.activeGameObject == null)
        {
            Debug.LogWarning("No GameObject selected to paste to!");
            return;
        }

        GameObject target = Selection.activeGameObject;

        foreach (Component copiedComponent in copiedComponents)
        {
            Component newComponent = target.AddComponent(copiedComponent.GetType());

            // Copy all serialized properties
            SerializedObject sourceObject = new SerializedObject(copiedComponent);
            SerializedObject targetObject = new SerializedObject(newComponent);
            SerializedProperty property = sourceObject.GetIterator();

            while (property.NextVisible(true))
            {
                targetObject.CopyFromSerializedProperty(property);
            }

            targetObject.ApplyModifiedProperties();
        }

        Debug.Log($"Pasted {copiedComponents.Count} components to {target.name}");
    }
}
```
