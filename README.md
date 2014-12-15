MLMI
====
clc;
clear;

% [V,info] = ReadData3D('VSD.Liver.XX.XX.OT.26929.mha');
% info = mha_read_header('VSD.Liver.XX.XX.OT.26929.mha');
% V = mha_read_volume('VSD.Liver.XX.XX.OT.26929.mha');


% preprocessing
Img = imread('shape1.png');
Img = rgb2gray(Img);
h = fspecial('laplacian');
Img = imfilter(Img,h);
% imshow(Img);
n = nnz(Img);
boundry = zeros(n,2);
cnt = 0;
for i = 1 : 493
    for j = 1 : 712
        if Img(i,j) > 0
            cnt = cnt+1;
            boundry(cnt,:) = [i,j];
        end
    end
end
% Img = Img - edge;
imshow(Img);
hold on
% [X,Y] = ginput(5);
% X = [367;312;203;439;124];
% Y = [163;162;170;178;247];
X = [505;444;371;290;148;125;115;123;119;139];
Y = [190;224;272;324;385;356;317;253;268;217];
Pos = [X,Y];
[m,~] = size(Pos);
plot(Pos(:,1),Pos(:,2),'dy');
% Pos_Update = zeros(5,2);
% plot(Pos(:,1),Pos(:,2),'dr');
% hold off
% figure;
% imshow(Img);
% % hold on
% 
%    plot(Img(Pos(1,1)),Img(Pos(1,2)),'dr'); 
%   
%     plot(Img(Pos(2,1)),Img(Pos(2,2)),'dr'); 
%  
% 
%     plot(Img(Pos(3,1)),Img(Pos(3,2)),'db'); 
%    
% 
%     plot(Img(Pos(4,1)),Img(Pos(4,2)),'db'); 
%  
% 
%     plot(Img(Pos(5,1)),Img(Pos(5,2)),'db'); 
%    
% 

% R = 3*sigma;


% defining distance matrix

D = zeros(m,m);
for i = 1 : m
    for j = i : m
        if i == j
            D(i,j) = 0;
        else
        D(i,j) = distance(Pos(i,1:2),Pos(j,1:2));
        D(j,i) = D(i,j);
        end
    end
end
cnt = 0;
indeces =zeros(1,m);
force =zeros(m,2);
boundry_dist = zeros(n,1);
R_collect = zeros (1,m);
for t = 1 : 1000
    i = 1;
    R = 1;
while i <= m
    
    for j = 1 : m
        if D(i,j) <= R 
            if D(i,j) > 0
                indeces(1,cnt+1) = j;
                force(j,1) = 1/Pos(j,1) + 1/Pos(j,1);
                force(j,2) = 1/Pos(j,2) + 1/Pos(j,2);
                cnt = cnt +1;
            end
        end
    end
    
%     find proper radius
    if cnt <= 1
        R = R+1;
        i = i-1;
        cnt = 0;
        indeces(1,1:m) = 0;
        forces(1:m,1:2) = 0;
%  flag = 1 -->   proper radius found
    else
        flag = 1;
        R_collect(1,i) = R;
    end
    % standard deviation
    sigma = sqrt(var(Pos));

    if flag == 1 
%         if R < (3*sigma)
            angle = zeros(1,cnt);
            for z = 1 : cnt
                angle(1,z) = atand(Pos(indeces(1,z),2)/Pos(indeces(1,z),1));
            end
            temp = 1;
            alpha = 0;
            q = zeros(1,2);
            
            while temp <= cnt
                alpha = abs(angle(1,temp) - angle(1,temp+1));
                if alpha < 90
                    q(1,1) = force(indeces(1,temp), 1) + force(indeces(1,temp+1),1);
                    q(1,2) = force(indeces(1,temp), 2) + force(indeces(1,temp+1),2);
                else if alpha == 90
                        q(1,1) = force(indeces(1,temp), 1);
                        q(1,2) = force(indeces(1,temp+1),2);
                    else if alpha > 90 && alpha < 180
                            q(1,1) = force(indeces(1,temp), 1) - force(indeces(1,temp+1),1);
                            q(1,2) = force(indeces(1,temp), 2) + force(indeces(1,temp+1),2);
                        else if alpha == 180
                                q(1,1) = force(indeces(1,temp), 1) - force(indeces(1,temp+1),1);
                                q(1,2) = 0;
                            else if alpha > 180 && alpha < 270
                                    q(1,1) = force(indeces(1,temp), 1) - force(indeces(1,temp+1),1);
                                    q(1,2) = force(indeces(1,temp), 2) - force(indeces(1,temp+1),2);
                                else if alpha == 270
                                        q(1,1) = force(indeces(1,temp), 1);
                                        q(1,2) = -force(indeces(1,temp+1),2);
                                    else if alpha > 270 && alpha < 360
                                            q(1,1) = force(indeces(1,temp), 1) + force(indeces(1,temp+1),1);
                                            q(1,2) = force(indeces(1,temp), 2) - force(indeces(1,temp+1),2);
                                        else
                                            q(1,1) = force(indeces(1,temp),1) + force(indeces(1,temp+1),1);
                                            q(1,2) = 0;
                                        end
                                    end
                                end
                            end
                        end
                    end
                end
                angle(1,temp) = alpha;
                force = [force(1:m,:);q];
                indeces(1,temp) = m+temp;
                for l = temp+2 : cnt
                    angle(1,l-1) = angle(1,l);
                    indeces(1,l-1) = indeces(1,l);
                end
                cnt = cnt-1;
                if cnt == 1
                    Pos(i,:) = Pos (i,:)+force(indeces(1,temp),:);
                    indeces(1,:) = 0;
                    angle(1,:) = 0;
                    cnt = 0;
                    flag = 0;
                    break;
                end
            end
%         end
    end

    i = i+1;
%     if i > 5
%         R_flag = 0;
%         for o = 1 : 5
%             if R_flag > 3*sigma
%                 
end
    f =t/1000;
    if f == 1 || f == 2 || f==3 || f==4 || f==5 || f== 6|| f == 7 || f==8 || f==9 || f == 10
        for b = 1 : m
            boundry_dist(:,1) = 0;
            for u = 1 : n
                boundry_dist(u,1) = distance(Pos(b,:),boundry(u,:)); 
            end
            [~,Index] = min(boundry_dist);
            Pos(i,:) = boundry(Index,:);
%             plot(Pos(i,2),Pos(i,1),'dg');
        end
    end
end

% plot(Pos(:,1),Pos(:,2),'dy');

% boundry_dist = zeros(n,1);
% for i = 1 : m
%     for u = 1 : n
%         boundry_dist(u,1) = distance(Pos(i,:),boundry(u,:)); 
%     end
%     [~,Index] = min(boundry_dist);
%     Pos(i,:) = boundry(Index,:);
% end
plot(boundry(:,2),boundry(:,1),'.');
plot(Pos(1:10,2),Pos(1:10,1),'dg');

