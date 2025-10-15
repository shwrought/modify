--// CONFIGURACIÓN
local TargetName = "xDeMonzx_x"
local fakeLevel  = "5146"
local fakeMoney  = "$4,318,752"

--// SERVICIOS
local Players = game:GetService("Players")

-- Función para modificar directamente la UI del leaderboard
local function spoofUI()
    -- Nota: acceder a CoreGui puede fallar por permisos (CoreGui está protegido).
    local success, PlayerList = pcall(function()
        return game:GetService("CoreGui"):WaitForChild("PlayerList")
    end)
    if not success or not PlayerList then
        warn("[spoofUI] No se pudo acceder a CoreGui.PlayerList (protegido).")
        return
    end

    -- Obtener todos los descendientes del PlayerList
    local children = PlayerList:GetDescendants()

    for _, obj in pairs(children) do
        -- Buscar el TextLabel que contiene el nombre objetivo
        if obj:IsA("TextLabel") and string.find(obj.Text, TargetName) then
            -- Suponemos que la estructura es: TextLabel (nombre) -> parent -> row (contiene columnas)
            local row = obj.Parent and obj.Parent.Parent
            if not row then
                continue
            end

            -- Recorremos las columnas (siblings) de la fila para cambiar Level y Money
            for _, col in pairs(row:GetChildren()) do
                if not col:IsA("TextLabel") then
                    continue
                end

                -- Detectar y reemplazar Level (número, longitud pequeña)
                if string.find(col.Text, "%d") and string.len(col.Text) < 6 then
                    col.Text = fakeLevel
                end

                -- Detectar y reemplazar Money (contiene $ o dígitos)
                if string.find(col.Text, "%$") or string.find(col.Text, "%d") then
                    col.Text = fakeMoney
                end
            end
        end
    end
end

-- Reaplicar cada pocos segundos para que no se resetee
while task.wait(2) do
    spoofUI()
end
