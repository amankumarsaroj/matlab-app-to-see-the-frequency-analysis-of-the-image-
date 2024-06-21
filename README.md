# matlab-app-to-see-the-frequency-analysis-of-the-image-
classdef My_Project_FOH < matlab.apps.AppBase

    % Properties that correspond to app components
    properties (Access = public)
        UIFigure                      matlab.ui.Figure
        ImageURLEditField             matlab.ui.control.EditField
        ImageURLEditFieldLabel        matlab.ui.control.Label
        FiltersDropDown               matlab.ui.control.DropDown
        FiltersDropDownLabel          matlab.ui.control.Label
        BandWidthEditField            matlab.ui.control.NumericEditField
        BandWidthEditFieldLabel       matlab.ui.control.Label
        Slider_2                      matlab.ui.control.Slider
        Slider                        matlab.ui.control.Slider
        RadiusEditField               matlab.ui.control.NumericEditField
        RadiusEditFieldLabel          matlab.ui.control.Label
        SpatialFrequencyFiltersLabel  matlab.ui.control.Label
        LoadImageButton               matlab.ui.control.Button
        ApplyFilterButton             matlab.ui.control.Button
        FilterMaskButton              matlab.ui.control.Button
        Image                         matlab.ui.control.Image
        UIAxesFilteredImage           matlab.ui.control.UIAxes
        UIAxesFilterMask              matlab.ui.control.UIAxes
    end

    
    properties (Access = private)
        I =  imread('Fourier.jpg'); % Variable to store image
        Mask % Filter msak matrix
        RDouble % Description
        GDouble % Description
        BDouble % Description
    end
    
    methods (Access = private)
        
        function LPfilter = LowpassFilter(~,r,s)
            X = linspace(-1, 1, s(2));
            Y = linspace(-1, 1, s(1));
            [x,y] = meshgrid(X,Y);
            radius = sqrt(x.^2 + y.^2);
            LPfilter = radius <= r;
        end
        function HPfilter = HighpassFilter(~,r,s)
            X = linspace(-1, 1, s(2));
            Y = linspace(-1, 1, s(1));
            [x,y] = meshgrid(X,Y);
            radius = sqrt(x.^2 + y.^2);
            HPfilter = radius >= r;
        end
        function BPfilter = BandpassFilter(~,r,r1,s)
            X = linspace(-1, 1, s(2));
            Y = linspace(-1, 1, s(1));
            [x,y] = meshgrid(X,Y);
            radius = sqrt(x.^2 + y.^2);
            BPfilter = radius >= r & radius <= r1;
        end
    end
    

    % Callbacks that handle component events
    methods (Access = private)

        % Button pushed function: FilterMaskButton
        function FilterMaskButtonPushed(app, event)
                 % Load the image                
                 [R,G,B] = imsplit(app.I);
                 app.RDouble = im2double(R);
                 app.GDouble = im2double(G);
                 app.BDouble = im2double(B);
                 r = app.RadiusEditField.Value;
                 r1 = app.BandWidthEditField.Value;
                 if strcmp(app.FiltersDropDown.Value,'Low pass')
                     app.Mask = app.LowpassFilter(r,size(app.I));

                 elseif strcmp(app.FiltersDropDown.Value,'High pass')
                     app.Mask = app.HighpassFilter(r,size(app.I));

                 elseif strcmp(app.FiltersDropDown.Value,'Band pass')
                    app. Mask = app.BandpassFilter(r,(r1+r),size(app.I));

                 end 
                     % Display the filter mask
                     imagesc(app.UIAxesFilterMask, app.Mask);
                     axis(app.UIAxesFilterMask,'image')
        end

        % Button pushed function: ApplyFilterButton
        function ApplyFilterButtonPushed(app, event)
            IfRed = fftshift(fft2(app.RDouble));
            IfGreen = fftshift(fft2(app.GDouble));
            IfBlue = fftshift(fft2(app.BDouble));
            imageWithFilterRed = IfRed .* app.Mask;
            imageWithFilterGreen = IfGreen .* app.Mask;
            imageWithFilterBlue = IfBlue .* app.Mask;
            imageWithFilter = cat(3,  imageWithFilterRed,imageWithFilterGreen, imageWithFilterBlue );
            realImage =  ifft2(imageWithFilter);
            imagesc(app.UIAxesFilteredImage,abs(realImage))
            axis (app.UIAxesFilteredImage,'image')
        end

        % Button pushed function: LoadImageButton
        function LoadImageButtonPushed(app, event)
             
            app.I = webread(app.ImageURLEditField.Value);
            app.Image.ImageSource = app.I;
           
        end

        % Value changed function: RadiusEditField
        function RadiusEditFieldValueChanged(app, event)
            app.Slider.Value  = app.RadiusEditField.Value;
        end

        % Value changing function: Slider
        function SliderValueChanging(app, event)
            app.RadiusEditField.Value = event.Value;
        end

        % Value changed function: BandWidthEditField
        function BandWidthEditFieldValueChanged(app, event)
            app.Slider_2.Value = app.BandWidthEditField.Value;
        end

        % Value changing function: Slider_2
        function Slider_2ValueChanging(app, event)
           app.BandWidthEditField.Value = event.Value;
        end

        % Value changed function: FiltersDropDown
        function FiltersDropDownValueChanged(app, event)
            value = app.FiltersDropDown.Value;
            
            if strcmp(value, 'Low pass')
                set(app.BandWidthEditField, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
                set(app.BandWidthEditFieldLabel, 'Enable', 'Off');
            elseif strcmp(value, 'High pass')
                set(app.BandWidthEditField, 'Enable', 'Off');
                set(app.Slider_2, 'Enable', 'Off');
                set(app.BandWidthEditFieldLabel, 'Enable', 'Off');
            elseif strcmp(value, 'Band pass')
                set(app.BandWidthEditField, 'Enable', 'On');
                set(app.Slider_2, 'Enable', 'On');
                set(app.BandWidthEditFieldLabel, 'Enable', 'On');
            end
        end
    end

    % Component initialization
    methods (Access = private)

        % Create UIFigure and components
        function createComponents(app)

            % Create UIFigure and hide until all components are created
            app.UIFigure = uifigure('Visible', 'off');
            app.UIFigure.Position = [100 100 485 543];
            app.UIFigure.Name = 'MATLAB App';

            % Create UIAxesFilterMask
            app.UIAxesFilterMask = uiaxes(app.UIFigure);
            title(app.UIAxesFilterMask, 'Filter mask')
            app.UIAxesFilterMask.XTick = [];
            app.UIAxesFilterMask.YTick = [];
            app.UIAxesFilterMask.XGrid = 'on';
            app.UIAxesFilterMask.YGrid = 'on';
            app.UIAxesFilterMask.ZGrid = 'on';
            colormap(app.UIAxesFilterMask, 'hot')
            app.UIAxesFilterMask.Position = [231 250 218 195];

            % Create UIAxesFilteredImage
            app.UIAxesFilteredImage = uiaxes(app.UIFigure);
            title(app.UIAxesFilteredImage, 'Filtered Image')
            app.UIAxesFilteredImage.XTick = [];
            app.UIAxesFilteredImage.YTick = [];
            colormap(app.UIAxesFilteredImage, 'gray')
            app.UIAxesFilteredImage.Position = [231 19 218 207];

            % Create Image
            app.Image = uiimage(app.UIFigure);
            app.Image.Position = [6 365 112 120];
            app.Image.ImageSource = 'Fourier.jpg';

            % Create FilterMaskButton
            app.FilterMaskButton = uibutton(app.UIFigure, 'push');
            app.FilterMaskButton.ButtonPushedFcn = createCallbackFcn(app, @FilterMaskButtonPushed, true);
            app.FilterMaskButton.Position = [42 111 100 22];
            app.FilterMaskButton.Text = 'Filter Mask';

            % Create ApplyFilterButton
            app.ApplyFilterButton = uibutton(app.UIFigure, 'push');
            app.ApplyFilterButton.ButtonPushedFcn = createCallbackFcn(app, @ApplyFilterButtonPushed, true);
            app.ApplyFilterButton.Position = [42 46 100 22];
            app.ApplyFilterButton.Text = 'Apply Filter';

            % Create LoadImageButton
            app.LoadImageButton = uibutton(app.UIFigure, 'push');
            app.LoadImageButton.ButtonPushedFcn = createCallbackFcn(app, @LoadImageButtonPushed, true);
            app.LoadImageButton.Position = [360 490 78 22];
            app.LoadImageButton.Text = 'Load Image';

            % Create SpatialFrequencyFiltersLabel
            app.SpatialFrequencyFiltersLabel = uilabel(app.UIFigure);
            app.SpatialFrequencyFiltersLabel.HorizontalAlignment = 'center';
            app.SpatialFrequencyFiltersLabel.FontSize = 18;
            app.SpatialFrequencyFiltersLabel.FontWeight = 'bold';
            app.SpatialFrequencyFiltersLabel.Position = [6 517 472 23];
            app.SpatialFrequencyFiltersLabel.Text = 'Spatial Frequency Filters';

            % Create RadiusEditFieldLabel
            app.RadiusEditFieldLabel = uilabel(app.UIFigure);
            app.RadiusEditFieldLabel.Position = [6 313 67 22];
            app.RadiusEditFieldLabel.Text = 'Radius ';

            % Create RadiusEditField
            app.RadiusEditField = uieditfield(app.UIFigure, 'numeric');
            app.RadiusEditField.Limits = [0 1];
            app.RadiusEditField.ValueDisplayFormat = '%0.3g';
            app.RadiusEditField.ValueChangedFcn = createCallbackFcn(app, @RadiusEditFieldValueChanged, true);
            app.RadiusEditField.Position = [78 313 100 22];

            % Create Slider
            app.Slider = uislider(app.UIFigure);
            app.Slider.Limits = [0 1];
            app.Slider.ValueChangingFcn = createCallbackFcn(app, @SliderValueChanging, true);
            app.Slider.Position = [12 299 160 3];

            % Create Slider_2
            app.Slider_2 = uislider(app.UIFigure);
            app.Slider_2.Limits = [0 1];
            app.Slider_2.ValueChangingFcn = createCallbackFcn(app, @Slider_2ValueChanging, true);
            app.Slider_2.Position = [12 204 160 3];

            % Create BandWidthEditFieldLabel
            app.BandWidthEditFieldLabel = uilabel(app.UIFigure);
            app.BandWidthEditFieldLabel.Position = [6 217 67 25];
            app.BandWidthEditFieldLabel.Text = 'Band Width';

            % Create BandWidthEditField
            app.BandWidthEditField = uieditfield(app.UIFigure, 'numeric');
            app.BandWidthEditField.Limits = [0 1];
            app.BandWidthEditField.ValueDisplayFormat = '%0.3g';
            app.BandWidthEditField.ValueChangedFcn = createCallbackFcn(app, @BandWidthEditFieldValueChanged, true);
            app.BandWidthEditField.Position = [78 217 100 25];

            % Create FiltersDropDownLabel
            app.FiltersDropDownLabel = uilabel(app.UIFigure);
            app.FiltersDropDownLabel.HorizontalAlignment = 'center';
            app.FiltersDropDownLabel.Position = [165 433 38 22];
            app.FiltersDropDownLabel.Text = 'Filters';

            % Create FiltersDropDown
            app.FiltersDropDown = uidropdown(app.UIFigure);
            app.FiltersDropDown.Items = {'No Filter', 'Low pass', 'High pass', 'Band pass'};
            app.FiltersDropDown.ValueChangedFcn = createCallbackFcn(app, @FiltersDropDownValueChanged, true);
            app.FiltersDropDown.Position = [134 407 98 22];
            app.FiltersDropDown.Value = 'No Filter';

            % Create ImageURLEditFieldLabel
            app.ImageURLEditFieldLabel = uilabel(app.UIFigure);
            app.ImageURLEditFieldLabel.HorizontalAlignment = 'right';
            app.ImageURLEditFieldLabel.Position = [6 490 67 22];
            app.ImageURLEditFieldLabel.Text = 'Image URL';

            % Create ImageURLEditField
            app.ImageURLEditField = uieditfield(app.UIFigure, 'text');
            app.ImageURLEditField.Position = [117 490 244 22];

            % Show the figure after all components are created
            app.UIFigure.Visible = 'on';
        end
    end

    % App creation and deletion
    methods (Access = public)

        % Construct app
        function app = My_Project_FOH

            % Create UIFigure and components
            createComponents(app)

            % Register the app with App Designer
            registerApp(app, app.UIFigure)

            if nargout == 0
                clear app
            end
        end

        % Code that executes before app deletion
        function delete(app)

            % Delete UIFigure when app is deleted
            delete(app.UIFigure)
        end
    end
end
