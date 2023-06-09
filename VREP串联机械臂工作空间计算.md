# VREP串联机械臂工作空间计算

```lua
function sysCall_init()
        
    -- Put your initialization code here初始化
    dlg=sim.displayDialog('Workspace','Computing reachable points.&&nThis may take a few seconds...',sim.dlgstyle_message,false,nil)
    a=0
    
end
doCalculation=function()
    local divisions=5 -- the divisions used for each joint. The total nb of pts becomes (divisions+1)^7
    local checkCollision=true
    local usePointCloud=true -- if false, 3D spheres will be used
    local extractConvexHull=true
    local generateRandom=true -- Use random configurations instead of joint divisions
    local randomSteps=46000 -- Number of random configs to test


    local jointHandles={}
    local jointLimitLows={}
    local jointLimitRanges={}
    local initialJointPositions={}
    for i=1,7,1 do
        jointHandles[i]=sim.getObjectHandle('LBR_iiwa_7_R800_joint'..i)
        local cyclic,interv=sim.getJointInterval(jointHandles[i])
        if cyclic then
            jointLimitLows[i]=-180*math.pi/180
            jointLimitRanges[i]=360*math.pi/180
        else
            jointLimitLows[i]=interv[1]
            jointLimitRanges[i]=interv[2]
        end
        initialJointPositions[i]=sim.getJointPosition(jointHandles[i])
    end
    local tip=sim.getObjectHandle('LBR_iiwa_7_R800_tip')
    local base=sim.getObjectHandle('LBR_iiwa_7_R800')
    local robotCollection=sim.getCollectionHandle('LBR_iiwa_7_R800')
    local points={}
    local normals={}

    if not usePointCloud then
        pointContainer=sim.addDrawingObject(sim.drawing_spherepoints,0.01,0.01,base,99999,{1,1,1})
    end
    
    if generateRandom then
        for cnt=1,randomSteps,1 do
            local p1=jointLimitLows[1]+math.random()*jointLimitRanges[1]
            sim.setJointPosition(jointHandles[1],p1)
            local p2=jointLimitLows[2]+math.random()*jointLimitRanges[2]
            sim.setJointPosition(jointHandles[2],p2)
            local p3=jointLimitLows[3]+math.random()*jointLimitRanges[3]
            sim.setJointPosition(jointHandles[3],p3)
            local p4=jointLimitLows[4]+math.random()*jointLimitRanges[4]
            sim.setJointPosition(jointHandles[4],p4)
            local p5=jointLimitLows[5]+math.random()*jointLimitRanges[5]
            sim.setJointPosition(jointHandles[5],p5)
            local p6=jointLimitLows[6]+math.random()*jointLimitRanges[6]
            sim.setJointPosition(jointHandles[6],p6)
            local p7=jointLimitLows[7]+math.random()*jointLimitRanges[7]
            sim.setJointPosition(jointHandles[7],p7)

            local colliding=false
            local matrix=sim.getObjectMatrix(tip,-1)
            -- local pos=sim.getObjectPosition(tip,-1)
            if checkCollision then
                if sim.checkCollision(robotCollection,sim.handle_all)~=0 then
                    colliding=true
                end
            end
            if not colliding then
                points[#points+1]=matrix[4]
                points[#points+1]=matrix[8]
                points[#points+1]=matrix[12]
                if usePointCloud then
                    normals[#normals+1]=matrix[3]
                    normals[#normals+1]=matrix[7]
                    normals[#normals+1]=matrix[11]
                else
                    sim.addDrawingObjectItem(pointContainer,{matrix[4],matrix[8],matrix[12]})
                end
            end
        end
    else
        for i1=1,divisions+1,1 do
            local p1=jointLimitLows[1]+(i1-1)*jointLimitRanges[1]/divisions
            sim.setJointPosition(jointHandles[1],p1)
            for i2=1,divisions+1,1 do
                local p2=jointLimitLows[2]+(i2-1)*jointLimitRanges[2]/divisions
                sim.setJointPosition(jointHandles[2],p2)
                for i3=1,divisions+1,1 do
                    local p3=jointLimitLows[3]+(i3-1)*jointLimitRanges[3]/divisions
                    sim.setJointPosition(jointHandles[3],p3)
                    for i4=1,divisions+1,1 do
                        local p4=jointLimitLows[4]+(i4-1)*jointLimitRanges[4]/divisions
                        sim.setJointPosition(jointHandles[4],p4)
                        for i5=1,divisions+1,1 do
                            local p5=jointLimitLows[5]+(i5-1)*jointLimitRanges[5]/divisions
                            sim.setJointPosition(jointHandles[5],p5)
                            for i6=1,divisions+1,1 do
                                local p6=jointLimitLows[6]+(i6-1)*jointLimitRanges[6]/divisions
                                sim.setJointPosition(jointHandles[6],p6)
                                for i7=1,divisions+1,1 do
                                    local p7=jointLimitLows[7]+(i7-1)*jointLimitRanges[7]/divisions
                                    sim.setJointPosition(jointHandles[7],p7)
                                
                                    local colliding=false
                                    local matrix=sim.getObjectMatrix(tip,-1)
                                    -- local pos=sim.getObjectPosition(tip,-1)
                                    if checkCollision then
                                        if sim.checkCollision(robotCollection,sim.handle_all)~=0 then
                                            colliding=true
                                        end
                                    end
                                    if not colliding then
                                        points[#points+1]=matrix[4]
                                        points[#points+1]=matrix[8]
                                        points[#points+1]=matrix[12]
                                        if usePointCloud then
                                            normals[#normals+1]=matrix[3]
                                            normals[#normals+1]=matrix[7]
                                            normals[#normals+1]=matrix[11]
                                        else
                                            sim.addDrawingObjectItem(pointContainer,{matrix[4],matrix[8],matrix[12]})
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
            end
        end
    end


    for i=1,7,1 do
        sim.setJointPosition(jointHandles[i],initialJointPositions[i])
    end

    if usePointCloud then
        sim.addPointCloud(0,255,base,0,5,points,{255,255,255,0,0,0,64,64,64,0,0,0},nil,normals)
    end

    if extractConvexHull then
        local vertices,indices=sim.getQHull(points)
        local shape=sim.createMeshShape(3,0,vertices,indices)
        sim.reorientShapeBoundingBox(shape,-1)
        sim.setShapeColor(shape,nil,0,{1,0,1})
        sim.setShapeColor(shape,nil,4,{0.2})
        sim.setObjectName(shape,'ConvexHull')
    end
end



function sysCall_actuation()
    -- Put your main ACTUATION code here
    a=a+1
    if a==2 then
        doCalculation()
        sim.endDialog(dlg)
    end
end


function sysCall_sensing()
    -- Put your main SENSING code here
end


function sysCall_cleanup()
    -- Put some restoration code here
end

```

